# Spectral Function Plot
## Overview

This repository contains scripts and instructions for performing Density Functional Theory (DFT) calculations using Questaal, and for processing and visualizing the results using python libraries. This workflow is designed for researchers in our group to optimize, use, and contribute to.
## Prerequisites

    Linux operating system or a compatible environment that can run shell scripts and the Questaal suite.
    Python 3.x with the following packages installed:
        pandas for data manipulation.
        matplotlib for data visualization.
    Questaal installed and properly configured on your system.
    An understanding of DFT calculations and the specific requirements for your project.

## Installation and Setup

    Clone this repository to your local machine using:
```sh
git clone https://github.com/bistad/Spectral_function_plot.git
```
    
    Ensure that Questaal is installed and correctly configured on your system. Refer to the official Questaal documentation for  installation instructions.
    Install the required Python packages by running pip install pandas matplotlib.

## Usage
**Running DFT(Questaal) Calculations using python**

Prepare your `script.sh` file with the appropriate Questaal commands for your DFT calculations, input file ctrl.extension(ctrl.fept in our example), and syml.fept file with information on symmetry points. Assuming these three files in the same folder, run the following command in the terminal to submit your job to the system:

```python
import subprocess
import time
import os

command = ['sbatch', 'script.sh']
result = subprocess.run(command, capture_output=True, text=True)

# Note: The calculation will produce an output file named spf.fept, which will be used for further processing.
```
**Data Processing:Part-1**

Preprocess the spf.fept file to extract arrays of symmetry points (syml_val) and generate the output.csv file. This step might require a custom script based on your specific needs.
Use the following Python snippet to import the output.csv file, preprocess the data, and calculate the average spectral function:

```python

# Get the current working directory path
cwd = os.getcwd()

# Check if the file exists in the current directory
if os.path.exists(os.path.join(cwd, 'spf.fept')):
    with open('spf.fept', 'r') as file:
        lines = file.readlines()

    with open('first_line.dat', 'w') as file:
        file.write(lines[0])
    with open('first_line.dat','r') as f:
        content = f.read()
        content = content.replace('#', '').strip()
        file_string = content.split()
        syml_val  = [0] + [float(num) for num in file_string]
```
Then the following code snippet is used to convert spf.fept without the first line into output.csv file.
```python
input_file_path = 'spf.fept'

with open(input_file_path, 'r') as file:
    lines = file.readlines()
with open(input_file_path, 'w') as file:
    file.writelines(lines[1:])

# Path to the new CSV file
output_file_path = 'output.csv'

# Open the original file and read the lines
with open(input_file_path, 'r') as infile, open(output_file_path, 'w', newline='') as csvfile:
    # Assuming each line of the file needs to be converted
    writer = csv.writer(csvfile)
    
    # Process each line in the original file
    for line in infile:
        # Assuming the original file might have comments or empty lines
        if line.split() and not line.startswith('#'):
            values = line.strip().split()
              # Write the values as a row in the CSV file
            writer.writerow(values)
```
**Data Processing:Part-2**

Then,output.csv is imported as dataframe and preprocessed further like taking average between up and down spectral functions
, converting energy to eV, and removing unnecessary columns and relabelling averaged column.
```python
df = pd.read_csv('output.csv', names= ['frequency','distance','A_up', 'A_down'])
df['A_up'] = (df['A_up'] + df['A_down'])/2
df['frequency']=df['frequency'].apply(lambda x: x*13.60569806)
df = df.drop(columns=['A_down'])
df = df.rename(columns={'A_up': 'A'})
df.head()
```

**Visualization**

To visualize the spectral function, use the following Python code as a template. Adjust the plotting parameters as necessary to suit your data:

```python
import matplotlib.pyplot as plt
import pandas as pd

def plot_spectral_function(df,filename=None):
    """
    Plot the spectral function.
    :param df: pandas DataFrame with the data.
    """
    fig, ax = plt.subplots(figsize=(8, 5))
    sc = ax.scatter(df['distance'], df['frequency'], c=df['A'], vmin=0, vmax=100,cmap='cividis', rasterized=True)
    cbar = fig.colorbar(sc, ax=ax)
    cbar.ax.tick_params(labelsize=15)
    array = syml_val
    labels = ['X','$\Gamma$','M','X','R','A','$\Gamma$','Z']
    ax.set_ylabel('Energy (eV)',fontsize=20)
    ax.set_xticks(array)
    ax.set_xticklabels(labels,fontsize=20)
    ax.grid(axis="x",color='white')
    plt.yticks(fontsize=20)
    ax.tick_params(axis='y', direction='in',size=15)
    ax.tick_params(axis='x', direction='in')
    ax.set_xlim(0,3.75804)
    ax.set_ylim(-3.5,2.5)
    plt.axhline(y=0, xmin=0, xmax=3.75804,color='white',linestyle="--")
    plt.tight_layout()
    plt.title("Spectral function of L1$_0$ phase FePt")
    if filename:
        plt.savefig(filename, format="png")
    plt.show()
    
plot_spectral_function(df,filename="fept.png")
```
This script will generate a plot of the spectral function, showing the relationship between distance (along symmetry points), energy, and the average spectral function through the broadening of the plot.

Contributing

We welcome contributions from everyone, especially from members of our research group! If you have suggestions for improvements or new features, please feel free to fork the repository, make your changes, and submit a pull request. For major changes, please open an issue first to discuss what you would like to change.

License

MIT
