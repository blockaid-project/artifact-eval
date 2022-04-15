# Artifact Evaluation README
**Paper**: Blockaid: Data Access Policy Enforcement for Web Applications (OSDI '22)

[PDF version of this README](https://raw.githubusercontent.com/blockaid-project/artifact-eval/main/Blockaid%20Artifact%20Evaluation%20README.pdf).

(The submission title was Embargo: Data Access Policy Enforcement for Web Applications. The submission used the system name “Embargo” for blinding, and the system is referred to as Embargo in the comparison plots.)

## Getting Started Instructions
The artifact consists of a Docker image that:
* Launches six Amazon EC2 c4.8xlarge instances to run experiments in parallel.
* Generates Figure 5, Table 2, and Figure 8 from the paper.
* Produces a PDF that compares the generated figures with those from the paper.

### Amazon EC2
To be able to launch Amazon EC2 instances, the artifact requires AWS credentials with appropriate EC2 permissions (e.g., `AmazonEC2FullAccess` suffices). Please note down your AWS Access Key and Secret Key for later use.

By default, the artifact will launch EC2 instances in the **us-east-2** (Ohio) region from our public image `ami-01457e9b6b7cdee4e`, which contains Blockaid and the web applications under evaluation. If you would like to launch the experiments in another region, please [copy](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/CopyingAMIs.html) the image to the desired region and note down its AMI ID. Alternatively, [import](https://docs.aws.amazon.com/vm-import/latest/userguide/vmimport-image-import.html) this VM image (6.5 GB) as an AMI in your desired region: https://blockaid-ae.s3.us-east-2.amazonaws.com/export-ami-0d21f4fd779d80014.vmdk.

### Test Run
On a machine with [Docker installed](https://docs.docker.com/get-docker/), create a directory to store the output of a test run:
```
mkdir /home/ubuntu/blockaid_test
```
Download and launch the Docker image  with the directory mounted (prepend with sudo if necessary):
```
docker run -it --rm --name blockaid_test --mount \
  type=bind,source=/home/ubuntu/blockaid_test,target=/data \
  blockaid/ae:buildx-latest
```
You should now see a command prompt:
```
root@9bc6f02e8a8c:/app#
```
Start the experiment script in “test mode”, which measures a small number of iterations and should finish in roughly **25 minutes**:
```
root@9bc6f02e8a8c:/app# ./run_all.sh test
You will be prompted for your AWS credentials and other information:
AWS credentials not found...
AWS Access Key ID: xxxx ↵
AWS Secret Access Key: xxxx ↵
AWS region [default: us-east-2]: ↵
EC2 AMI ID [default: ami-01457e9b6b7cdee4e]: ↵
```

(If you need to later modify your AWS credentials, you can do so by manually editing the file `.credentials.sh` using the nano text editor, or by deleting `.credentials.sh` and re-running `run_all.sh`.)

The script will now launch the test experiment on EC2, gather the results, delete the AWS resources, and produce a report file in the output directory named `all_plots.pdf`. You may view the PDF the host machine (i.e., outside the container).

(If the script fails or is killed mid-way, you can manually clean up the AWS resources created for the experiment by calling `./cleanup.sh` within the container.)

Inspect the report for comparisons between the figures generated from the experiment and figures from the submission. Because the test run has a shorter warmup phase, we expect the test report to indicate _higher latency across the board_ than reported in the paper.

Exit the Docker container by pressing Ctrl-D.

### Detailed Instructions
The artifact supports three modes, each taking a different amount of time:

| Mode      | Expected duration |
| ----------- | ----------- |
| `test`      | 25 min       |
| `small`   | 1 hr 40 min        |
| `full`      | 15 hr       |

When starting an experiment, you can pass the mode name as a command line argument to `run_all.sh`, like we did with the `test` mode just now. The `full` mode corresponds to the experiment setup reported in the paper submission, and the `small` mode allows an experiment that is less accurate but faster than the full version.

You can choose to run either the `small` or the `full` experiment. Here is an example for running the small experiment:
```
mkdir /home/ubuntu/blockaid_small
docker run -it --rm --name blockaid_test --mount \
  type=bind,source=/home/ubuntu/blockaid_small,target=/data \
  blockaid/ae:buildx-latest
root@9bc6f02e8a8c:/app# ./run_all.sh small
AWS credentials not found...
AWS Access Key ID: xxxx ↵
AWS Secret Access Key: xxxx ↵
AWS region [default: us-east-2]: ↵
EC2 AMI ID [default: ami-01457e9b6b7cdee4e]: ↵
...
```
Once again, inspect the report at `all_plots.pdf` in the output directory. The smaller the run, the higher the measured latencies are expected to be due to fewer warmup iterations. However, for the small experiment (and, of course, the full experiment) we do expect the relative latencies to match what is reported in the submission – e.g., the relative positions of points in Figure 5. (The same cannot be said of the test experiment, which is too short to yield stable measurements.)
