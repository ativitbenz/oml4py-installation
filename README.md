# oml4py-installation
Original document:   
```
https://docs.oracle.com/en/database/oracle/machine-learning/oml4py/1/mlpug/oracle-machine-learning-python.html#GUID-D00976CA-3663-4F32-A6A2-B6BF5A843ADC
```

<details>
<summary>Topic</summary>
  
- [OML4Py On Premises System Requirements](#System-Requirements)
- [Build and Install Python for Linux for On-Premises Databases]
- [Install the Required Supporting Packages for Linux for On-Premises Databases]
- [Install OML4Py Server for On-Premises Oracle Database]
- [Install OML4Py Client for On-Premises Databases]
- [Install Jupyter notebook and configuration](#setup-jupyter-notebook)
</details>

## System-Requirements
`note`: OML4Py on premises runs on 64-bit platforms only.

Both client and server on-premises components are supported on the Linux platforms listed in the table below.

On-Premises OML4Py Platform Requirements
Operating System and Hardware Platform	Description
  - Oracle Linux x86-64 7.x 64-bit 
  - Oracle Linux x86-64 8.x 64-bit 

On-Premises OML4Py Configuration Requirements and Server Support Matrix
  - Oracle Machine Learning for Python Version	Python Version	On-Premises Oracle Database Release 1.0	3.9.5	19c, 21c

setp1: Build and Install Python for Linux for On-Premises Databases   
1.Go to the Python website and download the Gzipped source tarball. The downloaded file name is Python-3.9.5.tgz  
`wget https://www.python.org/ftp/python/3.9.5/Python-3.9.5.tgz`   

2.Create a directory $ORACLE_HOME/python and extract the contents to this directory  
`mkdir -p $ORACLE_HOME/python
tar -xvzf Python-3.9.5.tgz --strip-components=1 -C $ORACLE_HOME/python`
The contents of the Gzipped source tarball will be copied directly to $ORACLE_HOME/python   

3.Go to the new directory   
`cd $ORACLE_HOME/python`

4.OML4Py requires the presence of the perl-Env, libffi-devel, openssl, openssl-devel, tk-devel, xz-devel, zlib-devel, bzip2-devel, readline-devel, libuuid-devel and ncurses-devel libraries.
You can confirm that those libraries are present by issuing the following commands:   
```
rpm -qa perl-Env
rpm -qa libffi-devel
rpm -qa openssl 
rpm -qa openssl-devel
rpm -qa tk-devel
rpm -qa xz-devel
rpm -qa zlib-devel
rpm -qa bzip2-devel
rpm -qa readline-devel
rpm -qa libuuid-devel
rpm -qa ncurses-devel
rpm -qa sqlite-devel
```
If the libraries are present, then those commands should return messages such as the following. Depending on the version of Linux that you are using, such as version 7.3 or 7.5, the exact messages differ slightly.   
```
perl-Env-1.04-2.el7.noarch
libffi-devel-3.0.13-19.el7.i686
libffi-devel-3.0.13-19.el7.x86_64
openssl-devel-1.0.2k-19.0.1.el7.x86_64
tk-devel-8.5.13-6.el7.i686
xz-devel-5.2.2-1.el7.x86_64
zlib-devel-1.2.7-17.el7.x86_64
zlib-devel-1.2.7-17.el7.i686
bzip2-devel-1.0.6-13.el7.x86_64
bzip2-devel-1.0.6-13.el7.i686
readline-devel-6.2-11.el7.i686
readline-devel-6.2-11.el7.x86_64
libuuid-devel-2.23.2-61.el7_7.1.x86_64
ncurses-devel-5.9-14.20130511.el7_4.x86_64
```   
The actual value returned depends on the version of Linux that you are using.  
If no output is returned, then install the packages as sudo or root user.  
`sudo yum install perl-Env libffi-devel openssl openssl-devel tk-devel xz-devel zlib-devel bzip2-devel readline-devel libuuid-devel ncurses-devel`  

5.To build Python 3.9.5, enter the following commands, where PREFIX is the directory in which you installed Python-3.9.5. The command on the Oracle Machine Learning for Python server will be:   
```
cd $ORACLE_HOME/python
./configure --enable-shared --enable-loadable-sqlite-extensions --prefix=$ORACLE_HOME/python

make clean; make
make altinstall
```  
  `note` Note:Be sure to use the --enable-shared flag if you are going to use Embedded Python Execution; otherwise, using an Embedded Python Execution function results in an extproc error.
Be sure to invoke make altinstall instead of make install to avoid overwriting the system Python.  

6.Set environment variable PYTHONHOME and add it to your PATH, and set environment variable LD_LIBRARY_PATH:   
```
export PYTHONHOME=$ORACLE_HOME/python
export PATH=$PYTHONHOME/bin:$PATH
export LD_LIBRARY_PATH=$PYTHONHOME/lib:$LD_LIBRARY_PATH
```
  `note` Note:In order to use Python for OML4Py, the variables must be set, and these variables must appear before system Python in PATH and LD_LIBRARY_PATH.     
pip will return warnings during package installation if the latest version is not installed. You can upgrade the version of pip to avoid these warnings:   
`python3 -m pip install --upgrade pip`

7.Create a symbolic link in your $ORACLE_HOME/python/bin directory to link to your python3.9 executable, which you can do with the following commands:  
```
cd $ORACLE_HOME/python/bin
ln -s python3.9 python3
```   
You can now start Python by running the command python3. To verify the directory where Python is installed, use the sys.executable command from the sys package. For example:   
```
$ python3
Python 3.9.5 (default, Feb 22 2022, 15:13:36)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-44.0.3)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> print(sys.executable)
/u01/app/oracle/product/19.3/dbhome_1/python/bin/python3
```
This returns the absolute path of the Python executable binary.  
  
If you run the command python3 and you get the error command not found, then that means the system cannot find an executable named python3 in $PYTHONHOME/bin. A symlink is required for the OML4Py server installation components. So, in that case, you need to create a symbolic link in your PREFIX/bin directory to link to your python3.9 executable as described in Step 6.   


# Install the Required Supporting Packages for Linux for On-Premises Databases
Both the OML4Py server and client installations for an on-premises Oracle database require that you also install a set of supporting Python packages, as described below.   

Installing required packages on OML4Py client machine   

The on-premises OML4Py client requires the following Python packages:  

numpy 1.21.5   
pandas 1.3.4  
scipy 1.7.3  
cx_Oracle 8.1.0  
scikit-learn 1.0.1  
matplotlib 3.3.3  
Use pip3.9 to install the supporting packages. For OML4Py client installation of all the packages, run the following command, specifying the package: ex.pip3.9 install packagename
```
pip3.9 install pandas==1.3.4
pip3.9 install scipy==1.7.3
pip3.9 install matplotlib==3.3.3
pip3.9 install cx_Oracle==8.1.0
pip3.9 install threadpoolctl==2.1.0
pip3.9 install joblib==0.14.0
pip3.9 install scikit-learn==1.0.1 --no-deps
pip3.9 uninstall numpy
pip3.9 install numpy==1.21.5
```
This command installs the cx_Oracle package using an example proxy server:   
` pip3.9 install cx_Oracle==8.1.0 --proxy="http://www-proxy.example.com:80"` 
## Installing required packages on OML4Py server machine  

On the OML4Py server machine, all these packages must be installed into $ORACLE_HOME/oml4py/modules so they can be detected by the Embedded Python Execution process. Run the following command, specifying the package and target directory, $ORACLE_HOME/oml4py/modules:  ex.pip3.9 install packagename --target=$ORACLE_HOME/oml4py/modules
```
pip3.9 install pandas==1.3.4 --target=$ORACLE_HOME/oml4py/modules
pip3.9 install scipy==1.7.3 --target=$ORACLE_HOME/oml4py/modules
pip3.9 install matplotlib==3.3.3 --target=$ORACLE_HOME/oml4py/modules
pip3.9 install cx_Oracle==8.1.0 --target=$ORACLE_HOME/oml4py/modules
pip3.9 install threadpoolctl==2.1.0 --target=$ORACLE_HOME/oml4py/modules
pip3.9 install joblib==0.14.0 --target=$ORACLE_HOME/oml4py/modules
pip3.9 install scikit-learn==1.0.1 --no-deps --target=$ORACLE_HOME/oml4py/modules
pip3.9 uninstall numpy 
pip3.9 install numpy==1.21.5 --target=$ORACLE_HOME/oml4py/modules
```
This command installs the cx_Oracle package using an example proxy server:  
```pip3.9 install cx_Oracle==8.1.0 --proxy="http://www-proxy.example.com:80" --target=$ORACLE_HOME/oml4py/modules```

Verify the Package Installation  
Load the packages below to ensure they have been installed successfully. Start Python and run the following commands:
```
$ python3
Python 3.9.5 (default, Feb 22 2022, 15:13:36)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-44.0.3)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import numpy
>>> import pandas
>>> import scipy
>>> import matplotlib
>>> import cx_Oracle
>>> import sklearn
```
##  setup-jupyter-notebook
`note` If you want to connect browser on public IP 
  - Step1: generate the file by typing this line in console
    ```
    jupyter notebook --generate-config
    ```
  - Step2: edit the values
  
  ```
    vi  <your path>/jupyter_notebook_config.py
  ```
( add the following two line anywhere because the default values are commented anyway)
```
    c.NotebookApp.allow_origin = '*' #allow all origins
    c.NotebookApp.ip = '0.0.0.0' # listen on all IPs
```
  - Step3: once you closed the gedit, in case your port is blocked
```
sudo ufw allow 8888 # enable your tcp:8888 port, which is ur default jupyter port
```
  - Step4: set a password (option)
```
jupyter notebook password # it will prompt for password
```
  - Step5: start jupyter -> more option running notebook (https://docs.jupyter.org/en/latest/running.html)
```
jupyter notebook --no-browser
```
`and connect like http://xxx.xxx.xxx.xxx:8888/login?`


