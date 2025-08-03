# Unified Verification and Validation for Vision-Based Autonomous Systems

This repository provides a comprehensive pipeline for the verification and validation of vision-based autonomous systems featured in the paper ``Towards Unified Probabilistic Verification and Validation of Vision-Based Autonomy". The pipeline is demonstrated on two case studies: the classic Mountain Car environment and a synthetic benchmark system. The project is designed for maximal reproducibility, featuring fully available source code, standalone smoke tests for individual pipeline components, automated ablation studies to replicate the experiments presented in the manuscript, and is configured for Docker or standalone deployment.

## Project Structure

- **[`mountain-car`](mountain-car )**  
  This folder contains scripts and models specific to the Mountain Car environment. It includes:
  - **Data Generation**: Scripts to collect trajectories and estimation data.
  - **Training**: Code for training a DQN policy and a vision-based state estimator.
  - **Validation**: Scripts to validate design-time abstractions against test-time trajectories.
  - **PRISM Model Generation**: Tools to generate PRISM models for formal verification.
  - **Ablations**: Automated scripts to reproduce the ablation studies presented in the paper.
  - **Pretrained Models and Data**: Sample data, trained weights, and generated PRISM models.

- **[`synthetic`](synthetic )**  
  This folder mirrors the structure of the [`mountain-car`](mountain-car ) directory but is tailored for a synthetic benchmark system. It includes:
  - Scripts for data generation, interval computation, PRISM model generation, and validation.
  - Ablation study scripts to explore safety and verification tradeoffs.
  - Pre-generated data and models for quick evaluation.

- **`data/`**  
  Contains sample data files, including trajectories, confidence intervals, and other intermediate results used in the experiments. Additionally, the `ablation-data/` folder within `data/` isolates the models and data specifically used for the ablation studies.

- **`prism-models/`**  
  Stores PRISM model files and property specifications for both the Mountain Car and synthetic systems. These files are used for formal verification tasks.

- **`sample-weights/`**  
  Includes pretrained weights for the vision-based state estimator and reinforcement learning policy.

- **[`Dockerfile`](Dockerfile )**  
  A Docker configuration file for setting up a reproducible environment to run the project.

## Requirements

### Standalone

To run the project without Docker, ensure the following dependencies are installed on your system:

1. **Python 3.10+**:
   Install Python 3.10 or later from your preferred package manager or [python.org](https://www.python.org/).

2. **Python Packages**:
   Install the required Python packages using pip:
   ```bash
   pip install numpy scipy torch gym matplotlib opencv-python gymnasium gym[classic_control]
   ```

3. **PRISM Model Checker**:
   Download and install the PRISM model checker:
   ```bash
   tar xfz prism-4.8.1-linux64-x86.tar.gz
   cd prism-4.8.1-linux64-x86
   ./install.sh
   ```

   Ensure the `prism` binary is accessible in your `PATH` or note its location for use in commands.

### Docker

The project is configured to be deployed using Docker for ease of setup and reproducibility. Follow these steps to build and run the Docker image:

1. **Build the Docker Image**:
   Navigate to the `artifact-1621/` directory and run the following command to build the Docker image:
   ```bash
   docker build -t artifact-1621 -f artifact-1621/Dockerfile .
   ```

2. **Run the Docker Container**:
   Once the image is built, you can run the container with:
   ```bash
   docker run -it artifact-1621
   ```

3. **Access the Container**:
   To access the container's bash shell for debugging or manual execution, use:
   ```bash
   docker run -it artifact-1621 /bin/bash
   ```

4. **Rebuild the Image After Changes**:
   If you make changes to the project files or Dockerfile, rebuild the image.

### Resources

This project requires moderate compute resources to run effectively. For most tasks, a modern CPU with at least 4 cores and 8 GB of RAM is sufficient. However, for training or running PRISM model checking with large models, it is recommended to have:

- **CPU**: Multi-core processor (4+ cores recommended)
- **RAM**: At least 16 GB of RAM for memory-intensive tasks like PRISM model checking
- **Disk Space**: At least 10 GB of free disk space for storing data, models, and intermediate files

If running in Docker, ensure the container has access to sufficient memory and CPU resources. All model sizes in the ablations will build under default PRISM memory allocations.

## Mountain Car Smoke Test

Most scripts in the Mountain Car project include their own standalone "smoke test" that can be executed independently. These standalone tests are located at the end of most scripts and allow users to verify the functionality of individual components in the pipeline.

### Standalone Smoke Tests
To run a standalone smoke test for a specific script, simply execute the script directly from your preferred IDE, terminal, or Docker container.

- **`plant`**: Outputs a random, continuous-space discrete-time trajectory of the Mountain Car.
  ```bash
  python artifact-1621/mountain-car/plant.py
  ```

- **`getData.py`**: Executes multiple trials of the Mountain Car and saves the trajectories to `data/sample-data`.
  ```bash
  python artifact-1621/mountain-car/getData.py
  ```

- **`genIntervals.py`**: Generates confidence intervals for the IMDP model and saves them to `data/sample-intervals`.
  ```bash
  python artifact-1621/mountain-car/genIntervals.py
  ```

- **`pagen.py`**: Generates a dictionary of mappings from all abstract state/estimate pairs to future states and simulates a sample trial.
  ```bash
  python artifact-1621/mountain-car/pagen.py
  ```

- **`prismgen.py`**: Generates a PRISM model based on the intervals and dynamics, and saves it to `prism-models/sample-model.pm`.
  ```bash
  python artifact-1621/mountain-car/prismgen.py
  ```
  Users are encouraged to inspect the resulting PRISM model in their preferred text editor, or from the Docker container terminal with:
  ```bash
  nano artifact-1621/mountain-car/prism-models/sample-model.pm
  ```

- **`validate.py`**: Validates the model on test data, and outputs various conformance confidence statistics.
  ```bash
  python artifact-1621/mountain-car/validate.py
  ```

Also note the availability of the PRISM model `prism-models/tested-imdp.pm` utilized in the manuscript.

### Modular Smoke Test
To run a modular verification/validation of the Mountain Car system, follow these steps by either executing the referenced scripts from an IDE or terminal:

1. **Collect test trajectories**:
   ```bash
   python artifact-1621/mountain-car/getData.py
   ```

2. **Generate confidence intervals**:
   ```bash
   python artifact-1621/mountain-car/genIntervals.py
   ```

3. **Generate the PRISM model**:
   ```bash
   python artifact-1621/mountain-car/prismgen.py
   ```

4. **Perform PRISM Model Checking**:
   Use the PRISM model checker in bash to verify the generated model:
   ```bash
   artifact-1621/prism_ws/bin/prism artifact-1621/mountain-car/prism-models/sample-model.pm artifact-1621/mountain-car/prism-models/mc-props.props
   ```

5. **Perform Model Validation**:
   Execute
   ```bash
   python artifact-1621/mountain-car/validate.py
   ```

## Mountain Car Ablations (Full Evaluation)

We provide scripts to reproduce the full results of the Mountain Car case study.

1. **Safety chance from various initial states**:
   - Script: `stateAblation.py`
   - Description: Evaluates the safety chances of the Mountain Car system from various initial states. It generates confidence intervals, creates a PRISM model for each initial state, and runs verification using the PRISM model checker. Results are summarized with the minimum safety chances for each initial state.
   - Run the script directly from an IDE or from the terminal (standalone or Docker bash):
     ```bash
     python artifact-1621/mountain-car/stateAblation.py
     ```

2. **Verification and Validation Tradeoff**:
   - Script: `alphaAblation.py`
   - Description: Explores the tradeoff between verification and validation by varying the confidence level (`alpha`) for interval generation. For each `alpha`, it generates confidence intervals, validates the model, creates a PRISM model, and runs verification. Outputs include safety chances and median confidence values for each `alpha`.
   - Run the script directly from an IDE or from the terminal (standalone or Docker bash):
     ```bash
     python artifact-1621/mountain-car/alphaAblation.py
     ```

## Synthetic Study Smoke Test

Most scripts in the synthetic project include their own standalone "smoke test" that can be executed independently. These standalone tests are located at the end of most scripts and allow users to verify the functionality of individual components in the pipeline.

### Standalone Smoke Tests
To run a standalone smoke test for a specific script, simply execute the script directly from your preferred IDE, terminal, or Docker container.

- **`plant`**: Outputs a random, continuous-space discrete-time  trajectory of the synthetic system.
  ```bash
  python artifact-1621/synthetic/plant.py
  ```

- **`getData.py`**: Executes multiple trials of the synthetic system and saves the trajectories to `data/sample-data`.
  ```bash
  python artifact-1621/synthetic/getData.py
  ```

- **`genIntervals.py`**: Generates confidence intervals for the IMDP model and saves them to `data/sample-intervals`.
  ```bash
  python artifact-1621/synthetic/genIntervals.py
  ```

- **`pagen.py`**: Generates a dictionary of mappings from all abstract state/estimate pairs to future states and simulates a sample trial.
  ```bash
  python artifact-1621/synthetic/pagen.py
  ```

- **`prismgen.py`**: Generates a PRISM model based on the intervals and dynamics, and saves it to `prism-models/sample-model.pm`.
  ```bash
  python artifact-1621/synthetic/prismgen.py
  ```
  Users are encouraged to inspect the resulting PRISM model in their preferred text editor, or from the Docker container terminal with:
  ```bash
  nano artifact-1621/synthetic/prism-models/sample-model.pm
  ```

- **`validate.py`**: Validates the model on test data, and outputs various conformance confidence statistics.
  ```bash
  python artifact-1621/synthetic/validate.py
  ```

Also note the availability of the PRISM model `prism-models/tested-imdp.pm` utilized in the manuscript.

### Modular Smoke Test
To run a modular verification/validation of the synthetic system, follow these steps by either executing the referenced scripts from an IDE or terminal:

1. **Collect test trajectories**:
   ```bash
   python artifact-1621/synthetic/getData.py
   ```

2. **Generate confidence intervals**:
   ```bash
   python artifact-1621/synthetic/genIntervals.py
   ```

3. **Generate the PRISM model**:
   ```bash
   python artifact-1621/synthetic/prismgen.py
   ```

4. **Perform PRISM Model Checking**:
   Use the PRISM model checker in bash to verify the generated model:
   ```bash
   artifact-1621/prism_ws/bin/prism artifact-1621/synthetic/prism-models/sample-model.pm artifact-1621/synthetic/prism-models/synth-props.props
   ```

5. **Perform Model Validation**:
   Execute
   ```bash
   python artifact-1621/synthetic/validate.py
   ```

## Synthetic System Ablations (Full Evaluation)

We provide scripts to reproduce the full results of the Mountain Car case study.

1. **Safety chance from various initial states**:
   - Script: `stateAblation.py`
   - Description: Evaluates the safety chances of the synthetic system from various initial states. It generates confidence intervals, creates a PRISM model for each initial state, and runs verification using the PRISM model checker. Results are summarized with the minimum safety chances for each initial state.
   - Run the script directly from an IDE or from the terminal (standalone or Docker bash):
     ```bash
     python artifact-1621/synthetic/stateAblation.py
     ```

2. **Verification and Validation Tradeoff**:
   - Script: `alphaAblation.py`
   - Description: Explores the tradeoff between verification and validation by varying the confidence level (`alpha`) for interval generation. For each `alpha`, it generates confidence intervals, validates the model, creates a PRISM model, and runs verification. Outputs include safety chances and median confidence values for each `alpha`.
   - Run the script directly from an IDE or from the terminal (standalone or Docker bash):
     ```bash
     python artifact-1621/synthetic/alphaAblation.py
     ```

3. **Effect of Distribution Shift on Validation**:
   - Script: `biasAblation.py`
   - Description: This script examines how varying the bias parameter in the system arguments affects validation results. It first generates a set of intervals with no bias and the standard covariance. Then, it loops through a list of biases, generates new test data for each bias, and validates the conformance confidence against the pre-generated intervals.
   - Run the script directly from an IDE or from the terminal (standalone or Docker bash):
     ```bash
     python artifact-1621/synthetic/biasAblation.py
     ```

## Acknowledgements

This project utilizes the Mountain Car environment from OpenAI Gym and incorporates a pre-trained Mountain Car vision model shared by Thomas Waite and Radoslav Ivanov. Additionally, the PRISM model checker (https://www.prismmodelchecker.org/) was utilized for formal verification tasks. The authors thank Alessandro Abate for a discussion of Bayesian validation of Markov models. 

This material is based on research sponsored by AFRL/RW under agreement number FA8651-24-1-0007 and the NSF CAREER award CNS-2440920. The U.S. Government is authorized to reproduce and distribute reprints for Governmental purposes notwithstanding any copyright notation thereon. The opinions, findings, views, or conclusions contained herein are those of the authors and should not be interpreted as representing the official policies or endorsements, either expressed or implied, of the DAF, AFRL, NSF, or the U.S. Government.

---

<sup>[1]</sup> Kwiatkowska, M., Norman, G., & Parker, D. (2011). PRISM 4.0: Verification of Probabilistic Real-time Systems. In Proc. 23rd International Conference on Computer Aided Verification (CAV’11), volume 6806 of LNCS, pages 585-591, Springer, 2011.

<sup>[2]</sup> Waite, T., Geng, Y., Turnquist, T., Ruchkin, I., & Ivanov, R. (2025). State-Dependent Conformal Perception Bounds for Neuro-Symbolic Verification of Autonomous Systems. ArXiv. https://arxiv.org/abs/2502.21308

For more details, see comments in the individual scripts.
