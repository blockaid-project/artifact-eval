# Artifact Evaluation README
**Paper**: [Blockaid: Data Access Policy Enforcement for Web Applications](https://www.usenix.org/conference/osdi22/presentation/zhang) (OSDI '22)

(The submission title was _Embargo: Data Access Policy Enforcement for Web Applications_. The submission used the system name “Embargo” for blinding, and the system is referred to as Embargo in the comparison plots.)

## Getting Started Instructions
The entry point to this artifact is a [Docker image](https://hub.docker.com/repository/docker/blockaid/ae), which contains scripts that:
* Launches Amazon EC2 instances on your behalf to run the experiments.
* Generates Figure 2, Table 2, and Figure 3 from the paper.
* Produces a PDF that compares the generated figures with those from the paper.

### Amazon EC2
The Blockaid experiments run on Amazon EC2 c4.8xlarge instances.
By default, the artifact launches six such instances (totaling 36×6=216 vCPUs) to run experiments in parallel, although it provides an option to launch fewer instances at a time (see the Test Run section).

To be able to launch EC2 instances, the artifact requires AWS credentials with appropriate EC2 permissions (e.g., `AmazonEC2FullAccess` suffices). Please note down your AWS Access Key and Secret Key for later use.

By default, the artifact will launch EC2 instances in the **us-east-2** (Ohio) region from our public image `ami-01457e9b6b7cdee4e`, which contains Blockaid and the web applications under evaluation. If you would like to launch the experiments in another region, please [copy](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/CopyingAMIs.html) the image to the desired region and note down its AMI ID. Alternatively, [download the image](https://github.com/blockaid-project/ae-vm-image) and [import it](https://docs.aws.amazon.com/vm-import/latest/userguide/vmimport-image-import.html) as an AMI in your desired region.

### Test Run
You can run this artifact on any machine with [Docker installed](https://docs.docker.com/get-docker/), but we recommend running it from an Amazon EC2 instance. (A small one like t2.micro will suffice, but make sure it has enough disk space—like 20GB—for Docker!)

Pull the latest version of the Docker image:
```
docker pull blockaid/ae:latest
```

Create a directory to store the output of a **test run**:
```
mkdir /home/ubuntu/blockaid_test
```
Launch the Docker image with the directory mounted:
```
docker run -it --rm --name blockaid_test --mount \
  type=bind,source=/home/ubuntu/blockaid_test,target=/data \
  blockaid/ae:latest
```
You should now see a command prompt:
```
root@9bc6f02e8a8c:/app#
```
Let’s start the experiment script in "test mode", which measures a small number of iterations.
By default, the script launches six c4.8xlarge instances in parallel:
```
root@9bc6f02e8a8c:/app# ./run_all.sh test
```
You can adjust the degree of instance parallelism using the `PARALLEL` environment variable.
For example, to run at most one EC2 instance at a time:
```
root@9bc6f02e8a8c:/app# PARALLEL=1 ./run_all.sh test
```
You will be prompted for your AWS credentials and other information:
```
AWS credentials not found...
AWS Access Key ID: xxxx ↵
AWS Secret Access Key: xxxx ↵
AWS region [default: us-east-2]: ↵
EC2 AMI ID [default: ami-01457e9b6b7cdee4e]: ↵
```

(If you need to later modify your AWS credentials, you can do so by manually editing the file `.credentials.sh` using the nano text editor, or by deleting `.credentials.sh` and re-running `run_all.sh`.)

The script will now launch the test experiment on EC2, gather the results, delete the AWS resources, and produce a report file in the output directory named `all_plots.pdf`.
This process should finish within **25 minutes** under the default configuration (i.e., full parallelism), and within 2 hours under sequential execution (i.e., `PARALLEL=1`).
You may view the PDF the host machine (i.e., outside the container).

(If the script fails or is terminated mid-way, you can manually clean up the AWS resources created for the experiment by calling `./cleanup.sh` within the container.)

Inspect the report for comparisons between the figures generated from the experiment and figures from the submission. Because the test run has a shorter warmup phase, we expect the test report to indicate _higher latency across the board_ than reported in the paper.

Exit the Docker container by pressing Ctrl-D.

### Detailed Instructions
The artifact supports three modes, each taking a different amount of time. The three modes differ only in the number of measurement rounds.

<table>
<thead>
  <tr>
    <th rowspan="2">Mode</th>
    <th colspan="5">Number of rounds</th>
    <th colspan="2">Expected duration</th>
  </tr>
  <tr>
    <th>Original</th>
    <th>Modified</th>
    <th>Cached</th>
    <th>Cold cache</th>
    <th>No cache</th>
    <th>PARALLEL=6 (default)</th>
    <th>PARALLEL=1</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>test</td>
    <td>3</td>
    <td>3</td>
    <td>3</td>
    <td>3</td>
    <td>3</td>
    <td>25 min</td>
    <td>2 hr</td>
  </tr>
  <tr>
    <td>small</td>
    <td>300</td>
    <td>300</td>
    <td>300</td>
    <td>10</td>
    <td>10</td>
    <td>1 hr 40 min</td>
    <td>8 hr</td>
  </tr>
  <tr>
    <td>full</td>
    <td>3000</td>
    <td>3000</td>
    <td>3000</td>
    <td>100</td>
    <td>100</td>
    <td>15 hr</td>
    <td></td>
  </tr>
</tbody>
</table>

When starting an experiment, you can pass the mode name as a command line argument to `run_all.sh`, like we did with the `test` mode just now. The `full` mode corresponds to the experiment setup reported in the paper submission, and the `small` mode allows an experiment that is less accurate but faster than the full version.

You can choose to run either the `small` or the `full` experiment. Here is an example for running the small experiment (again, you can limit instance parallelism by passing the `PARALLEL` environment variable to `run_all.sh`):
```
mkdir /home/ubuntu/blockaid_small
docker run -it --rm --name blockaid_test --mount \
  type=bind,source=/home/ubuntu/blockaid_small,target=/data \
  blockaid/ae:latest
root@9bc6f02e8a8c:/app# ./run_all.sh small
AWS credentials not found...
AWS Access Key ID: xxxx ↵
AWS Secret Access Key: xxxx ↵
AWS region [default: us-east-2]: ↵
EC2 AMI ID [default: ami-01457e9b6b7cdee4e]: ↵
...
```
Once again, inspect the report at `all_plots.pdf` in the output directory. The smaller the run, the higher the measured latencies are expected to be due to fewer warmup iterations. However, for the small experiment (and, of course, the full experiment) we do expect the relative latencies to match what is reported in the submission – e.g., the relative positions of points in Figure 2. (The same cannot be said of the test experiment, which is too short to yield stable measurements.)
