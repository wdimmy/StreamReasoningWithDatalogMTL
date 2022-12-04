# Stream Reasoning with DatalogMTL

This repository contains code and other related resources of our paper "Stream Reasoning with DatalogMTL".

<span id='overview'/>

### Overview:
* <a href='#introduction'>1. Introduction</a>
* <a href='#example'>2. A Stream Reasoning Example </a>
* <a href='#data'>3. Automatic Temporal Data Generation </a>
    * <a href='#lubm'>3.1. Temporal LUBM Benchmark</a>
      * <a href="#downloadlubm">3.1.1 Download LUBM generator</a>
      * <a href="#datalog">3.1.2 Obtain datalog facts</a>
      * <a href="#datalogmtl">3.1.3 Add punctual intervals</a>
    * <a href='#hackathon'>3.2 Hackathon Benchmark</a>
     
* <a href='#mock'>4. Mocking stream reasoning scenarios </a>

****

<span id='introduction'/>

#### 1. Introduction: <a href='#overview'>[Back to Top]</a>
In the paper we study stream reasoning in DatalogMTL—an extension of Datalog with metric temporal operators. 
In particular, we propose a sound and complete stream reasoning algorithm that is applicable to forward-propagating DatalogMTL 
programs, in which propagation of derived information towards past time points is precluded. Memory consumption in our generic 
algorithm depends both on the properties of the rule set and the input data stream; in particular, it depends on the distances
between timestamps occurring in data. This may be undesirable in certain practical scenarios since these distances can be very
small, in which case the algorithm may require large amounts of memory. To address this issue, we propose a second algorithm,
where the size of the required memory becomes independent on the timestamps in the data at the expense of disallowing punctual
intervals in the rule set.We have implemented our stream reasoning algorithms as an extension of the DatalogMTL reasoner MeTeoR
and tested it experimentally. The obtained results support the feasibility of our approach in practice.


****

<span id="example"/>

#### 2. A Stream Reasoning Example 

Consider a network where nodes are monitor- ing different signals. A node receiving readings for a signal with  sufficiently 
high frequency flags the signal by sending a message to all neighbouring nodes in the network, which will 
then also monitor the signal for a fixed period of time. The analyst is interested in tracking which nodes are monitoring
which signals at any point in time. This generic monitoring task involves analysing how information propagates recursively 
over time throughout the topology ofthe network, and can be captured by the following DatalogMTL rules:


Flag(x, z) <--  Monit(x,z) ,  ALWAYS[-4,0]SOMETIME[-2,0]Signal(z)

ALWAYS[-3,0]Monit(x,z) <-- Flag(y,z),  Connect(x,y) 

Rule (1) states that a node monitoring a signal will flag it whenever 
the signal has been received continuously over an interval of length at least 4 with gaps 
between consecutive readings of length at most 2. In turn, Rule (2) ensures that each signal z flagged by 
a node y, will be monitored in the future by all nodes x directly connected to y in the network for a period
of length at least 3.

<span id="data"/>

#### 3. Automatic Temporal Data Generation </a>

<span id="lubm"/>

##### 3.1 Lehigh University Benchmark (LUBM)

<span id="downloadlubm"/>

######  3.1.1 Download the LUBM data generator

You can download the data generator (UBA) from **SWAT Projects - Lehigh University Benchmark (LUBM)** [website](http://swat.cse.lehigh.edu/projects/lubm/). In particular,
we used [UBA1.7](http://swat.cse.lehigh.edu/projects/lubm/uba1.7.zip).

After downloading the  UBA1.7 package, you need to add package's path to CLASSPATH. For examole,

```shell
export CLASSPATH="$CLASSPATH:your package path"
```

<span id="datalog"/>

###### 3.1.2 Generate the owl files
```
==================
USAGES
==================

command:
   edu.lehigh.swat.bench.uba.Generator
      	[-univ <univ_num>]
	[-index <starting_index>]
	[-seed <seed>]
	[-daml]
	-onto <ontology_url>

options:
   -univ number of universities to generate; 1 by default
   -index starting index of the universities; 0 by default
   -seed seed used for random data generation; 0 by default
   -daml generate DAML+OIL data; OWL data by default
   -onto url of the univ-bench ontology
```

We found some naming and storage issues when using the above command provided 
by the official documentation. To provide a more user-friendly way, we 
wrote a script which can be directly used to generate required owl files
by passing some simple arguments. An example is shown as follows,

```python
from meteor_reasoner.datagenerator import generate_owl

univ_nume = 1 # input the number of universities you want to generate
dir_name = "./data" # input the directory path used for the generated owl files.

generate_owl.generate(univ_nume, dir_name)

```
In  **./data**, you should obtain a serial of owl files like below,
```
University0_0.owl 
University0_12.owl  
University0_1.owl
University0_4.owl
.....
```

Then, we need to convert the owl files to datalog-like facts. We also prepare
a script that can be directly used to do the conversion. 
```python
from meteor_reasoner.datagenerator import generate_datalog

owl_path = "owl_data" # input the dir_path where owl files locate
out_dir = "./output" # input the path for the converted datalog triplets

generate_datalog.extract_triplet(owl_path, out_dir)
```
In **./output**, you should see a **./output/owl_data**  containing data
in the form of
```
UndergraduateStudent(ID0)
undergraduateDegreeFrom(ID1,ID2)
takesCourse(ID3,ID4)
undergraduateDegreeFrom(ID5,ID6)
UndergraduateStudent(ID7)
name(ID8,ID9)
......
```
and **./output/statistics.txt**  containing the statistics information
about the converted datalog-like data in the form of
```
worksFor:540
ResearchGroup:224
....
AssistantProfessor:146
subOrganizationOf:239
headOf:15
FullProfessor:125
The number of unique entities:18092
The number of triplets:8604
```
<span id="datalogmtl"/>

###### 3.1.3 Add punctual intervals

Up to now, we only construct the atemporal data, so the final step will be adding temporal information
(intervals) to these atemporal data. In the stream reasoning scenario, we consider punctual intervals, namely,
the leftendpint equals to the right endpoint (e.g., A@[1,1]). To be more specific, assuming that we have a datalog-like 
dataset in **datalog/datalog_data.txt**,
if we want to create a dataset containing 10000 facts and each facts has at most 2 intervals, each of 
time points are randomly chosen from a range [0, 300], we can run he following command (remember to add **--min_val=0, --max_val=300, --punctual**). 
```shell

python add_intervals.py --datalog_file_path datalog/datalog_data.txt --factnum 10000 --intervalnum 2 --min_val 0 --max_val 300 --punctual 

```

In the **datalog/10000.txt**, there should be 10000 facts, each of which in the form P(a,b)@\varrho, and 
a sample of facts are shown as follows,
```
undergraduateDegreeFrom(ID1,ID2)@[7,7]
takesCourse(ID34,ID4)@[46,46]
undergraduateDegreeFrom(ID5,ID6)@[21,21]
name(ID18,ID9)@[22,22]
......
```
<span id="hackathon"/>

##### 3.2 Hackathon Benchmark

The Hackathon Challenge, organised at the [Stream Reasoning Workshop 2021](https://streamreasoning.org/events/srw2021), provides a stream generator 
together with several reasoning tasks. We considered the scenario in the challenge where input streams 
contain data from Eclipse Simulation of Urban MObility ([SUMO](https://www.eclipse.org/sumo/)) describing road vehicles in a traffic jam, and the task is to detect
vehicles that make a short stop (less than 5 seconds). A detailed description about the Stream Generator could be found
[here](https://github.com/patrik999/stream-reasoning-challenge). The main idea is to use a server to generate a stream of data and then stream the data to the
client through a WebSocket. The server is also acting as a web server to control the server through our provided REST API.


<span id="mock"/>

#### 4. Mocking stream reasoning scenarios

Given the generated  static temporal data, we mock the stream reasoning scenarios by writing a script to
read the static temporal data and then output a set of facts having the same punctual point each time. 

##### 4.1 Using MeTeoR to do the streaming reasoning 

```shell
python meteor_mock.py --datapath ./demo/traffic1.txt --rulepath ./programs/efficient_shortbreak.txt
--datasetname traffic_shortbreak_small --target ShortBreak
```
Some intermediate results will be saved in ./results/meteor_traffic_shortbreak_small.pkl, which is a dictionary containing
the following information:

```
window_size: a integer list recording the window_size at each rule application iteration after coalescing
window_size_raw: a integer list recording the window_size at each rule application iteration before coalescing
```
--------------------------------------------------------------------------------
##### 4.2 Using MeTeoR_Str to do the streaming reasoning

```shell
python sr_mock.py --datapath ./demo/traffic1.txt --rulepath ./programs/efficient_shortbreak.txt
--datasetname traffic_shortbreak_small --target ShortBreak
```
Some intermediate results will be saved in ./results/sr_traffic_shortbreak_small.pkl, which is a dictionary containing the
following information:
```
window_size: a integer list recording the window_size at each time point after coalescing
window_size_raw: a integer list recording the window_size  at each time point before coalescing
run_times: a float list recording the run time at each time point
```

--------------------------------------------------------------------------------
