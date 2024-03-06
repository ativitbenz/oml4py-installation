# oml4py-installation
Original document: https://docs.oracle.com/en/database/oracle/machine-learning/oml4py/1/mlpug/oracle-machine-learning-python.html#GUID-D00976CA-3663-4F32-A6A2-B6BF5A843ADC

`note`: OML4Py on premises runs on 64-bit platforms only.

Both client and server on-premises components are supported on the Linux platforms listed in the table below.

On-Premises OML4Py Platform Requirements
Operating System and Hardware Platform	Description
  - Oracle Linux x86-64 7.x 64-bit Oracle Linux Release 7
  - Oracle Linux x86-64 8.x 64-bit Oracle Linux Release 8

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
```
