---
title: "Transitioning from Anaconda to Miniconda"
datePublished: Sun Nov 12 2023 14:34:42 GMT+0000 (Coordinated Universal Time)
cuid: clovkucz6000509i65tfwbmg2
slug: transitioning-from-anaconda-to-miniconda
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1699799607198/1412558f-b5d3-4f63-8fa3-1934b063da50.png
tags: python, data-science, miniconda, anaconda

---

Python enthusiasts often turn to Anaconda for its convenience in setting up Python environments, especially in the data science domain. However, there is a sleeker alternative: Miniconda. 

In this post, we'll explore the benefits of transitioning from Anaconda to Miniconda and provide a step-by-step guide to help you make the switch.

## The Data Science Setup

Traditionally, kickstarting your data science journey involved downloading and installing the comprehensive [Anaconda distribution](https://www.anaconda.com/distribution/). This bundled package includes Python, over 200 packages, and additional tools like [Anaconda Navigator](https://docs.anaconda.com/anaconda/navigator/) and `qtconsole`.

## A Leaner Alternative: Miniconda

While Anaconda simplifies the initial setup, its hefty size (around 600MB download, over 2GB installed) and inclusion of numerous unnecessary packages may not align with everyone's needs. 

Enter Miniconda, a minimalist distribution that includes only Python, the essential conda tool, and a handful of core packages. With a download size of around 70MB (less than a tenth of Anaconda), Miniconda offers a more efficient and customizable data science environment. Any additional software needed can be easily installed later using conda.

## Understanding Conda

There are a number of ways to set up your system to do data science in Python. In this tutorial we will use a tool called [conda](https://docs.conda.io/projects/conda/en/latest/). Conda is both a package manager and environment manager.

In the context of Python, a **package manager** is a tool that installs, updates and removes third-party Python packages. A third-party Python package is any Python package that is not a part of the Python standard library. The vast ecosystem of data science related packages (e.g. [pandas](https://pandas.pydata.org/), [NumPy](https://numpy.org/), [Matplotlib](https://matplotlib.org/), [scikit-learn](https://scikit-learn.org/), [TensorFlow](https://www.tensorflow.org/g) and more) make Python a go-to language for data enthusiasts.

Another popular package manager is [pip](https://pypi.org/), which is the default package manager for new Python installations. In this tutorial we'll only touch on conda.

An **environment manager** creates an isolated container on your computer, known as virtual environments, with their own installation of Python and third-party packages. This isolation ensures independence from any other Python installation on your machine.

## Advantages of Migrating to Miniconda

1. **Efficient Resource Utilization** Anaconda comes preloaded with a lot of packages, some of which might be unnecessary for your specific workflows. Miniconda offers a minimalist approach, allowing you to install only what you need. This not only saves valuable storage space, but also ensures your system resources are dedicated to the tools and libraries you actually tend to use. Okay, in Anaconda you have the `conda clean --all` command, which basically cleans unused package cache and other files. However, Miniconda's smaller package footprint tends to be more efficient.
    
2. **Simplified Environment Management** The [Anaconda Navigator](https://docs.anaconda.com/free/navigator/index.html), a graphical-user-interface became obsolete by the speed and simplicity of the command prompt or terminal. Creating a new environment might seem intimidating when you are just starting. However, you will quickly switch to the command prompt or terminal, since almost all tutorials you'll follow will use the command prompt or terminal. Embracing the command prompt or terminal will not only enhance efficiency but also integrates smoothly with your prefferred code editor.
    
3. **Streamlined Deployment** Deploying your code on production servers can be a daunting task, especially when you are dealing with the unnecessary bloat from Anaconda. Miniconda simplifies the deployment process by allowing you to create virtual environments with only the essential packages, ensuring a clean and efficient runtime on production servers.

## Step-by-step Approach

### 1\. (Optional) Export Environment Files

If you want to recover environments from Anaconda, export your environment files using the following commands:

```bash
conda activate your_env
```

Next, export your environment to a `.yml` file.

On Windows:

```plaintext
conda env export | findstr /V "^prefix: " > your_env.yml
```

On macOS or Linux:

```bash
conda env export | grep -v "^prefix: " > your_env.yml
```

#### Note on "prefix"

Note that conda creates a "prefix" in the `.yml` file. In the context of Conda environments, the "prefix" refers to the location on your file system where the Conda environment is installed. The "prefix" line in the `your_env.yml` file specifies the path to the directory where the environment is created.

For instance:

```yaml
prefix: C:\Users\YourUsername\Anaconda3\envs\your_environment_name
```

This information is used when you or someone else later tries to recreate the Conda environment from the `your_env.yml` file. The presence of the "prefix" allows Conda to know where to create the environment.

When you export the environment using `conda env export`, you might want to exclude the "prefix" line from the exported YAML file if you plan to share the environment file across different systems or users. This is because the specific path may not be valid or appropriate on another system. That's why the `grep -v "^prefix: "` (or `findstr /V "^prefix: "`) is used — it removes the "prefix" line from the exported environment file.

### 2\. Get Rid of Anaconda

Uninstall Anaconda by following the instructions [here](https://docs.anaconda.com/free/anaconda/install/uninstall/).

### 3\. Install Miniconda

Find instructions on how to install Miniconda [here](https://docs.conda.io/projects/miniconda/en/latest/).

### 4\. Create Miniconda Environments

If you want a clean start, we recommend creating new enviroments. You can use the following command to create a new environment with the latest (at the time of writing) Python version:

```bash
cond create -n "your_env" python=3.12
```

Activate the environment with:

```bash
conda activate your_env
```

Alternatively, you can use the following command to recreate a predefined environment. Note that this will install all packages listed in the `.yml` file (potentially packages you don't need).

```bash
conda env create -f your_env.yml
```

If you want to specify a different install path than the default for your system (not related to 'prefix' in the `your_env.yml`), just use the `-p` flag followed by the required path. Note that we previously deleted the "prefix" line. However you can use this command if you want to install an environment file with the prefix line still in it.

```bash
conda env create -f your_env.yml -p /home/user/anaconda3/envs/env_name
```

### 5\. Install Third-party Packages

Finally, we can install some packages in our new or recovered environment. I tend to install the `pandas` in almost al my environments. Let's install this packages in our activated environment.

```bash
conda install pandas
```

---

Congratulations! You've successfully made the transition from Anaconda to Miniconda. Stay tuned for more!

