# DLI Lab Setup

### Start 5G Network Simulator

In this DLI you will use an Agentic Generative AI solution to solve a bandwidth allocation problem. The lab will consist of two different parts. In the first part, the lab will show you how to setup an open source 5G Network Lab consisting of the following parts:
- 5G Core Lab simulation by Open Air Interface: https://openairinterface.org/oai-5g-core-network-project/
- FlexRIC that will be connected to the gNodeB and will be used to change the bandwidth allocation for each slice in the gNodeB
- RAN Lab composed by a gNodeB and two Use Equipment simulation components from Open Air OAI Softmode: https://github.com/simula/openairinterface5g/blob/dreibh/simulamet-testbed/doc/RUNMODEM.md
- Traffic generation over the Open Air network simulator using Iperf Tool: https://iperf.fr/

The Lab setup will start with the initialization of the 5G Core Network, then we will set up the gNodeB and the RIC connecting both via the E1 protocol. We will attach two UEs to the 5G network, each UE1 will have its own slice as seen in the diagram. Once UEs are functional, we will use the Iperf tool to generate traffic. First we will set up the Iperf server on the OAI External Network connected by the User Plane Function UPF.  Then we will use the Iperf Client to generate traffic against the External Network using the UEs connection. 

![5G Lab](./5glab.png)

In this Jupyter Notebook we will set the lab for the experiment. In a separate Jupyter Notebook we will build the Agentic Workflow for the experiment.

## Installing Requirements
In this first cell we will install the requirements for the Blueprint and we will restart the kernel, so you will need to press "yes" to the window that will pop up

```python
%%capture
!sudo apt install -y iperf3
!pip install -r ../requirements.txt
import IPython
IPython.Application.instance().kernel.do_shutdown(True)
```

### Installing Flexric and gNodeB

This code will take between 7 and 8 minutes to run. It  will compile the ric and gNodeB components within the DLI environment. If you want to install this lab in your computer, you just need to download the DLI directory and execute this command. 

```python
%%capture
!chmod +x build_ric_oai_ne.sh
!./build_ric_oai_ne.sh
```

### OAI 5G Network 
To set up the 5G core Network funcitons we will use the docker compose comand. First, we will setup the standard network funciton for the core and then we will set up an additonal slice (slice 2) that will have its own SMF and UPF

```python
%%capture
!docker compose --progress=plain -f docker-compose-oai-cn-slice1.yaml up -d
import time
time.sleep(20)
!docker compose --progress=plain -f docker-compose-oai-cn-slice2.yaml up -d
```

### Start RIC

Then we will start the FlexRIC to be able to modify parameters in the gNodeB on an ad hoc basis

```python
import subprocess
import os

# Ensure the logs directory exists
os.makedirs("logs", exist_ok=True)

cmd = "./flexric/build/examples/ric/nearRT-RIC"
logfile = "logs/RIC.log"

# Open log file for writing and start the process
with open(logfile, "a") as log:
    process = subprocess.Popen(
        ["bash", "-c", f"stdbuf -oL {cmd}"],  # stdbuf ensures real-time logging
        stdout=log,
        stderr=subprocess.STDOUT,
        universal_newlines=True
    )

print(f"Process started with PID {process.pid}, logging to {logfile}.")
time.sleep(5)
```

<details>
  <summary>Trouble Shooting - Sample output</summary>
  
  If flexric starts successfully you should see a similar output in logs/RIC.log
  ```
  [UTIL]: Setting the config -c file to /usr/local/etc/flexric/flexric.conf
[UTIL]: Setting path -p for the shared libraries to /usr/local/lib/flexric/
[NEAR-RIC]: nearRT-RIC IP Address = 127.0.0.1, PORT = 36421
[NEAR-RIC]: Initializing 
[NEAR-RIC]: Loading SM ID = 142 with def = MAC_STATS_V0 
[NEAR-RIC]: Loading SM ID = 143 with def = RLC_STATS_V0 
[NEAR-RIC]: Loading SM ID = 3 with def = ORAN-E2SM-RC 
[NEAR-RIC]: Loading SM ID = 146 with def = TC_STATS_V0 
[NEAR-RIC]: Loading SM ID = 148 with def = GTP_STATS_V0 
[NEAR-RIC]: Loading SM ID = 145 with def = SLICE_STATS_V0 
[NEAR-RIC]: Loading SM ID = 2 with def = ORAN-E2SM-KPM 
[NEAR-RIC]: Loading SM ID = 144 with def = PDCP_STATS_V0 
[iApp]: Initializing ... 
[iApp]: nearRT-RIC IP Address = 127.0.0.1, PORT = 36422
[NEAR-RIC]: Initializing Task Manager with 2 threads 
[E2AP]: E2 SETUP-REQUEST rx from PLMN   1. 1 Node ID 3584 RAN type ngran_gNB
[NEAR-RIC]: Accepting RAN function ID 2 with def = ORAN-E2SM-KPM 
[NEAR-RIC]: Accepting RAN function ID 3 with def = ORAN-E2SM-RC 
[NEAR-RIC]: Accepting RAN function ID 142 with def = MAC_STATS_V0 
[NEAR-RIC]: Accepting RAN function ID 143 with def = RLC_STATS_V0 
[NEAR-RIC]: Accepting RAN function ID 144 with def = PDCP_STATS_V0 
[NEAR-RIC]: Accepting RAN function ID 145 with def = SLICE_STATS_V0 
[NEAR-RIC]: Accepting RAN function ID 146 with def = TC_STATS_V0 
[NEAR-RIC]: Accepting RAN function ID 148 with def = GTP_STATS_V0 
  ```

### Start the gNodeB
Then we will run the gNodeB using the OAI softmoden software

```python
import subprocess
import os

cmd = "sudo ./openairinterface5g/cmake_targets/ran_build/build/nr-softmodem -O ran-conf/gnb.conf --sa --rfsim -E --gNBs.[0].min_rxtxtime 6"
logfile = "logs/gNodeB.log"

env = os.environ.copy()
env["LD_LIBRARY_PATH"] = "."

# Open log file for writing and start the process
with open(logfile, "a") as log:
    process = subprocess.Popen(
        ["bash", "-c", f"stdbuf -oL {cmd}"],  # stdbuf ensures real-time logging
        stdout=log,
        stderr=subprocess.STDOUT,
        universal_newlines=True
    )

print(f"Process started with PID {process.pid}, logging to {logfile}.")
time.sleep(5)
```

### Initialize Bandwidth 50 50 
The gNodeB will have initially allocated 50% of its bandwidth to each of the slices. 

- Note : Incase you get `[E2AP]: Resending Setup Request after timeout`error, rerun the `Start RIC` cell.

***
Please check `./logs/RIC.log` to ensure that flexRIC is working. Incase you see a "failed" message, try rerunning the `Start RIC` cell above. 

Error message : 
```
nearRT-RIC: /dli/task/llm-slicing-5g-lab/flexric/src/lib/e2ap/v2_03/enc/e2ap_msg_enc_asn.c:3165: e2ap_enc_e42_setup_response_asn_pdu: Assertion `sr->len_e2_nodes_conn > 0 && "No global node conected??"' failed.
```

***

```python
!./change_rc_slice.sh 50 50
time.sleep(5)
```

### Start the UE
After we will create a User Equipment Simulator and attach it to the gNodeB. Following cell creates UE1 and UE2

```python
import os
import time
import subprocess
import logging
from typing import Optional, List

import colorlog

# Configure colored logging.
handler = colorlog.StreamHandler()
handler.setFormatter(colorlog.ColoredFormatter(
    fmt="%(log_color)s%(asctime)s %(levelname)s: %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
    log_colors={
        'DEBUG':    'blue',
        'INFO':     'green',
        'WARNING':  'yellow',
        'ERROR':    'red',
        'CRITICAL': 'red,bg_white',
    }
))
logger = colorlog.getLogger(__name__)
logger.addHandler(handler)
logger.setLevel(logging.DEBUG)
logger.propagate = False

def start_async_process(name: str, cmd: str, logfile: str) -> Optional[subprocess.Popen]:
    """
    Start an asynchronous process and log its status.

    Args:
        name: Name of the process.
        cmd: The command to run.
        logfile: Path to the log file.

    Returns:
        The subprocess.Popen object if the process started successfully;
        None otherwise.
    """
    logger.info("Starting process %s with command: %s", name, cmd)
    try:
        with open(logfile, "a") as log:
            process = subprocess.Popen(
                ["bash", "-c", f"stdbuf -oL {cmd}"],
                stdout=log,
                stderr=subprocess.STDOUT,
                universal_newlines=True,
            )
        if process.pid:
            logger.info("Process %s started with PID %s, logging to %s", name, process.pid, logfile)
            return process
        else:
            logger.error("Process %s did not start properly.", name)
            return None
    except Exception as e:
        logger.error("Failed to start process %s: %s", name, str(e))
        return None
```

```python
def start_ue(ue_id: str, config_file: str, namespace: str, server_addr: str, port: str = "106") -> Optional[subprocess.Popen]:
    """
    Start a UE process.

    Args:
        ue_id: Identifier for the UE (used in command arguments and logging).
        config_file: Path to the UE configuration file.
        namespace: The network namespace for the UE.
        server_addr: The server address for the UE.
        port: The port used by the UE (default is "106").

    Returns:
        The subprocess.Popen object if the process started successfully; None otherwise.
    """
    cmd = f"""
    sudo ./multi_ue.sh -c{ue_id} -e & 
    sleep 5
    sudo ip netns exec {namespace} bash -c '
        sudo LD_LIBRARY_PATH=. ./openairinterface5g/cmake_targets/ran_build/build/nr-uesoftmodem \\
        --rfsimulator.serveraddr {server_addr} -r {port} --numerology 1 --band 78 -C 3619200000 \\
        --rfsim --sa -O {config_file} -E
    '
    """
    logfile = f"logs/UE{ue_id}.log"
    print(cmd)
    return start_async_process(f"UE{ue_id}", cmd, logfile)

ue1_process = start_ue("1", "ran-conf/ue_1.conf", "ue1", "10.201.1.100")
ue2_process = start_ue("3", "ran-conf/ue_2.conf", "ue3", "10.203.1.100")
```

**Debugging tip: Print logs for sanity check**

```
tail -f logs/UE1.log
tail -f logs/UE3.log
```

### Start the Iperf Tool Server
Once the 5G Network Simulation is running we will start the simulation of traffic by using with tool Iperf. First we will create an Iperf Server that will be on the External Network connected via the User Plane Function. 

```python
def start_iperf(name: str, port: str) -> Optional[subprocess.Popen]:
    """
    Start an iPerf3 server process.

    Args:
        name: Identifier for the iPerf3 instance.
        port: Port on which the iPerf3 server should run.

    Returns:
        The subprocess.Popen object if started successfully; None otherwise.
    """
    cmd = f"docker exec -t oai-ext-dn iperf3 -s -p {port}"
    logfile = f"logs/docker_iperfserver_{name}.log"
    return start_async_process(f"iPerf_{name}", cmd, logfile)

iperf1_process = start_iperf("server1", "5201")
iperf2_process = start_iperf("server2", "5202")
```

## Start Traffic generator and insert records in the database
Once every element in The Network is up and running and the Iperf server is listening in the external network. We will run two iperf clients that will be generating traffic in both UEs. These scripts will generate udp traffic from the iperf server towards the UE and will alternate speeds 30M and 120M for 100 seconds. In the following cell we will

1. Run traffic generator to alternate between 30M and 120M
2. Insert UE1 and UE2 iperf logs into the Kinetica Database. Kinetica is a very fast, distributed, GPU-accelerated database with advanced analytics, visualization, geospatial, and machine learning functionality.

```python
# Generate traffic 

from datetime import datetime
import logging
import random
import re
import os
import subprocess
import threading
from typing import Dict, List, Pattern

from dotenv import load_dotenv, set_key
from english_words import get_english_words_set
import gpudb
from gpudb import GPUdb
from gpudb import GPUdbColumnProperty as cp
from gpudb import GPUdbRecordColumn as rc
from IPython.display import display, HTML


class IperfRecord:
    def __init__(
        self,
        record_id: str = "",
        record_ue: str = None,
        record_timestamp: str = "",
        record_stream: int = None,
        record_interval_start: float = None,
        record_interval_end: float = None,
        record_duration: float = 0.0,
        record_data_transferred: float = None,
        record_bitrate: float = None,
        record_jitter: float = None,
        record_lost_packets: int = None,
        record_total_packets: int = None,
        record_loss_percentage: float = None
    ) -> None:
        self.id = record_id
        self.ue = record_ue
        self.timestamp = record_timestamp
        self.stream = record_stream
        self.interval_start = record_interval_start
        self.interval_end = record_interval_end
        self.duration = record_duration
        self.data_transferred = record_data_transferred
        self.bitrate = record_bitrate
        self.jitter = record_jitter
        self.lost_packets = record_lost_packets
        self.total_packets = record_total_packets
        self.loss_percentage = record_loss_percentage

    def record_to_dict(self) -> Dict[str, str | float]:
        return {
            "id": self.id,
            "ue": self.ue,
            "timestamp": self.timestamp,
            "stream": self.stream,
            "interval_start": self.interval_start,
            "interval_end": self.interval_end,
            "duration": self.duration,
            "data_transferred": self.data_transferred,
            "bitrate": self.bitrate,
            "jitter": self.jitter,
            "lost_packets": self.lost_packets,
            "total_packets": self.total_packets,
            "loss_percentage": self.loss_percentage
        }


def convert_bandwidth(bw_str: str) -> int:
    """
    Converts a bandwidth string like "120M" or "30M" into an integer in bits per second.
    """
    if bw_str.endswith("M"):
        return int(bw_str[:-1]) * 1_000_000
    elif bw_str.endswith("K"):
        return int(bw_str[:-1]) * 1_000
    else:
        return int(bw_str)
```

```python
# Load environment variables and initialize Kinetica connection.
load_dotenv()
kdbc_options = GPUdb.Options()
kdbc_options.username = os.environ.get("KINETICA_USERNAME")
kdbc_options.password = os.environ.get("KINETICA_PASSWORD")
kdbc_options.disable_auto_discovery = True
word_list: List[str] = list(get_english_words_set(['web2'], lower=True))


def generate_random_table_name() -> str:
    # Generate a table name with the correct schema, four random underscore-separated words, and the string "_iperf_3_logs"
    fully_qualified_random_table_name: str = f"{os.environ.get('KINETICA_SCHEMA', 'nvidia_gtc_dli_2025')}." + "_".join(random.choices(word_list, k=4)) + "_iperf3_logs"
    
    # Set our environment variable in the .env file for use by the Agents.
    set_key(
        dotenv_path="./.env",
        key_to_set="IPERF3_RANDOM_TABLE_NAME",
        value_to_set=fully_qualified_random_table_name,
        quote_mode="always",
        export=False,
        encoding="utf-8"
    )
    return fully_qualified_random_table_name


kdbc: GPUdb = GPUdb(
    host=os.environ.get("KINETICA_HOST"),
    options=kdbc_options
)

iperf3_table_name: str = generate_random_table_name()
if kdbc.has_table(table_name=iperf3_table_name).table_exists:
    kdbc.clear_table(table_name=iperf3_table_name)

schema, table = iperf3_table_name.split('.')
url = f'https://demo72.kinetica.com/gadmin/#/table/{schema}/{table}'
user = os.environ.get("KINETICA_USERNAME")
password = os.environ.get("KINETICA_PASSWORD")
html_out = f'''
    <b>Kinetica Table:</b> <a target="_blank" href="{url}">{table}</a> </br>
    <b>User:</b> {user} </br>
    <b>Password:</b> {password}
'''
display(HTML(html_out))

schema: List[List[str]] = [
    ["id",               rc._ColumnType.STRING, cp.UUID,     cp.PRIMARY_KEY, cp.INIT_WITH_UUID],
    ["ue",               rc._ColumnType.STRING, cp.CHAR8,    cp.DICT],
    ["timestamp",        rc._ColumnType.STRING, cp.DATETIME, cp.INIT_WITH_NOW],
    ["stream",           rc._ColumnType.INT,    cp.INT8,     cp.DICT],
    ["interval_start",   rc._ColumnType.FLOAT],
    ["interval_end",     rc._ColumnType.FLOAT],
    ["duration",         rc._ColumnType.FLOAT],
    ["data_transferred", rc._ColumnType.FLOAT],
    ["bitrate",          rc._ColumnType.FLOAT],
    ["jitter",           rc._ColumnType.FLOAT],
    ["lost_packets",     rc._ColumnType.INT],
    ["total_packets",    rc._ColumnType.INT],
    ["loss_percentage",  rc._ColumnType.FLOAT]
]
kdbc_table = gpudb.GPUdbTable(
    _type=schema,
    name=iperf3_table_name,
    db=kdbc
)

# Precompiled regex pattern to parse iperf3 output.
filter_regex: str = (
    r'^\[ *([0-9]+)\] +([0-9]+\.[0-9]+)-([0-9]+\.[0-9]+) +sec +'
    r'([0-9\.]+) +MBytes +([0-9\.]+) +Mbits/sec +([0-9\.]+) +ms +'
    r'([0-9]+)/([0-9]+) +\(([0-9\.]+)%\)$'
)
pattern: Pattern[str] = re.compile(filter_regex)


def iperf_runner(
    namespace: str,
    ue_name: str,
    bind_host: str,
    server_host: str,
    udp_port: int,
    bandwidth: str,
    test_length_secs: int,
    kdbc_table: gpudb.GPUdbTable,
    pattern: Pattern[str],
    log_file: str
) -> None:
    """
    Runs iperf for one UE, parses and inserts records into Kinetica,
    and writes each record to a dedicated log file in the specified format.
    Exits when the iperf process finishes.

    :param namespace: The network namespace for the UE.
    :param ue_name: A label/identifier for the UE (e.g., "UE1").
    :param bind_host: IP address to bind to (iperf3 -B).
    :param server_host: The remote iperf3 server IP address.
    :param udp_port: The UDP port to use (iperf3 -p).
    :param bandwidth: The bandwidth limit (e.g. "30M" or "120M").
    :param test_length_secs: The test duration in seconds (iperf3 -t).
    :param kdbc_table: The Kinetica table object where we insert records.
    :param pattern: Precompiled regex to parse iperf3 output.
    :param log_file: Path to the log file for this iperf3 process (e.g. "logs/UE1_iperfc.log").
    """
    try:
        iperf_cmd = (
            f"stdbuf -oL iperf3 "
            f"-B {bind_host} "
            f"-c {server_host} "
            f"-p {udp_port} "
            f"-R -u "
            f"-b {bandwidth} "
            f"-t {test_length_secs}"
        )
        cmd = ["sudo", "ip", "netns", "exec", namespace, "bash", "-c", iperf_cmd]

        # Open a subprocess to run iperf3.
        with subprocess.Popen(
            cmd,
            stdout=subprocess.PIPE,
            stderr=subprocess.STDOUT,
            universal_newlines=True,
            bufsize=1
        ) as proc:
            for line in proc.stdout:
                line = line.strip()
                match = pattern.match(line)
                if match:
                    # Create an IperfRecord from the parsed line.
                    iperf_record = IperfRecord(
                        record_ue=ue_name,
                        record_stream=int(match.group(1)),
                        record_interval_start=float(match.group(2)),
                        record_interval_end=float(match.group(3)),
                        record_data_transferred=float(match.group(4)),
                        record_bitrate=float(match.group(5)),
                        record_jitter=float(match.group(6)),
                        record_lost_packets=int(match.group(7)),
                        record_total_packets=int(match.group(8)),
                        record_loss_percentage=float(match.group(9))
                    )
                    # Calculate duration.
                    iperf_record.duration = iperf_record.interval_end - iperf_record.interval_start

                    # Insert record into Kinetica.
                    record_dict = iperf_record.record_to_dict()
                    kdbc_table.insert_records(record_dict)
                    kdbc_table.flush_data_to_server()

                    # Write the raw iperf3 output line to the dedicated log file
                    # with the format: "[<UE>] [<timestamp>] <line>"
                    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
                    with open(log_file, "a") as f:
                        f.write(f"[{ue_name}] [{timestamp}] {line}\n")
    except Exception as e:
        print(f"Error running process: {e}")
    # When iperf3 ends, the function exits so that the main thread can handle any post-run activities.


bandwidth_ue1: str = "30M"
bandwidth_ue2: str = "120M"
bind_host_ue1: str = "12.1.1.2"
bind_host_ue2: str = "12.1.1.130"
server_host_ue1: str = "192.168.70.135"
server_host_ue2: str = "192.168.70.135"
udp_port_ue1: int = 5201
udp_port_ue2: int = 5202
test_length_secs: int = 100

test_iterations: int = 25 # Feel free to change this as you see fit to run the log generation for a longer period of time.
current_iteration: int = 0

while current_iteration < test_iterations:
    logger.info(f"""CURRENT ITERATION: {current_iteration}""")
    current_iteration += 1
    
    t1: threading.Thread = threading.Thread(
        target=iperf_runner,
        args=(
            "ue1",
            "UE1",
            bind_host_ue1,
            server_host_ue1,
            udp_port_ue1,
            bandwidth_ue1,
            test_length_secs,
            kdbc_table,
            pattern,
            "logs/UE1_iperfc.log"
        ),
        daemon=True
    )

    t2: threading.Thread = threading.Thread(
        target=iperf_runner,
        args=(
            "ue3",
            "UE3",
            bind_host_ue2,
            server_host_ue2,
            udp_port_ue2,
            bandwidth_ue2,
            test_length_secs,
            kdbc_table,
            pattern,
            "logs/UE2_iperfc.log"
        ),
        daemon=True
    )

    t1.start()
    t2.start()

    t1.join()
    t2.join()

    if bandwidth_ue1 == "30M":
        bandwidth_ue1 = "120M"
    else:
        bandwidth_ue1 = "30M"

    if bandwidth_ue2 == "30M":
        bandwidth_ue2 = "120M"
    else:
        bandwidth_ue2 = "30M"
```

Keep the above cell running! If you want to see what is happening in the background, you can:
1. Check out UE1, UE2 iperf logs to see how traffic generator works, and how it leads to packet loss:
    ```bash
    tail -f logs/UE1_iperfc.log
    tail -f logs/UE2_iperfc.log
    ```

2. Access Kinetica database, and see how wlogs are updated there real-time. Login with your username and password, mentioned in the .env file.

    - KINETICA_USERNAME="nvidia_gtc_2025"
    - KINETICA_PASSWORD="Kinetica123!"
    - KINETICA_SCHEMA="nvidia_gtc_dli_2025"
    - Table name: os.environment.get("IPERF3_RANDOM_TABLE_NAME")
    - URL : https://demo72.kinetica.com/gadmin/

This will be the final step setting up the 5G Lab for the Agentic Workflow.

![stop](./Stop2.jpg) 


