# Running ODBC on CBL-Mariner

## Recommended path

```
FROM  mcr.microsoft.com/cbl-mariner/base/core:2.0

ENV ACCEPT_EULA=Y

RUN tdnf install -y \
    glibc-iconv \
    libltdl \   
    unixODBC \
    msodbcsql18 
```

## Persistent storage method
### (THIS IS NOT RECOMMENDED)
Download the following packages from the CBL-Mariner repo
* https://packages.microsoft.com/cbl-mariner/2.0/prod/Microsoft/x86_64/Packages/m/msodbcsql18-18.2.1.1-1.x86_64.rpm
* https://packages.microsoft.com/cbl-mariner/2.0/prod/base/x86_64/Packages/u/unixODBC-2.3.9-2.cm2.x86_64.rpm
* https://packages.microsoft.com/cbl-mariner/2.0/prod/base/x86_64/Packages/l/libltdl-2.4.6-8.cm2.x86_64.rpm
* https://packages.microsoft.com/cbl-mariner/2.0/prod/base/x86_64/Packages/g/glibc-iconv-2.35-4.cm2.x86_64.rpm

Use a tool like ![7-zip](https://www.7-zip.org/) or ![rpm2cpio](https://www.man7.org/linux/man-pages/man8/rpm2cpio.8.html) to extract the contents of the above RPMs into a directory.

The rough file struction should look something like this:
```
/odbctest/
|_ etc
  |_ unixODBC
|_ opt
  |_ microsoft
    |_ msodbcsql18
|_ usr
  |_ bin
  |_ lib
  |_ lib64
  |_ share
```

You will then need environment variables to help make sure all the items are properly accessible
```
ODBCSYSINI=/odbctest/etc/unixODBC
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/odbctest/usr/bin
LD_LIBRARY_PATH=/odbctest/usr/lib:/odbctest/opt/microsoft/msodbcsql18/lib64:/odbctest/usr/lib64
ACCEPT_EULA=Y
GCONV_PATH=/odbctest/usr/lib64/gconv
```

Configure the odbc ini files...

odbcinst.ini:
```
[ODBC Driver 18 for SQL Server]
Description=Microsoft ODBC Driver 18 for SQL Server
Driver=/odbctest/opt/microsoft/msodbcsql18/lib64/libmsodbcsql-18.2.so.1.1
UsageCount=1
```

odbc.ini:
```
[MSQL]
Driver = ODBC Driver 18 for SQL Server
Server = tcp:127.0.0.1,1433
Servername = 127.0.0.1
Port = 1433
Encrypt = yes
Database = mydb
TrustServerCertificate = yes
```

Finally you should be able to connect with `isql -v MSQL username password`
