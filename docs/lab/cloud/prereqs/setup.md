# Set up your environment

This lab is written to work on MacOS and Linux.  

On most Linux distributions you will have *tar* and it will be fine.

On most MacOS systems the default *tar* command is less desirable than a version of tar provided by the GNU project, called *gtar*. *gtar* is preferred for the labs.

Follow these instructions on your prep system.  The environment variables set by these instructions are in effect only for the terminal session in which you enter the commands. If you finish the labs in one sitting within the same terminal session, you will only have to repeat these instructions once.  Otherwise, you can repeat these instructions as necessary.

!!! Note

	The following set of commands will set an environment variable that will be used throughout the labs. It will point to either *tar* or *gtar* or it will print an error message if it cannot find either one.

	``` bash
	if [[ -n "${LAB_TAR}" ]]; then
	for i in {1..6} ; do echo '******************' ; done
	echo ''
	echo "CAUTION: Prior value of LAB_TAR was ${LAB_TAR}"
	echo "   Take note of this value in the unlikely event it was"
	echo "   already set by another application on your system"
	echo "   in which case you may need to restore this value later"
	echo ''
	for i in {1..6} ; do echo '******************' ; done
	fi
	unset LAB_TAR && which tar 1>/dev/null 2>&1 && LAB_TAR=tar
	which gtar 1>/dev/null 2>&1 && LAB_TAR=gtar
	echo ''
	if [[ -z "${LAB_TAR}" ]]; then 
	echo "ERROR:  neither tar nor gtar was found" 
	else
	echo You will use ${LAB_TAR} for the labs
	fi
	```

The following set of commands will set an environment variable that will be used throughout the labs if the *base64* command supports the *wrap* option. (Typically, the wrap option will be supported on Linux systems but not on MacOS.)

``` bash
if [[ -n "${LAB_WRAP}" ]]; then
  for i in {1..6} ; do echo '******************' ; done
  echo ''
  echo "CAUTION: Prior value of LAB_WRAP was ${LAB_WRAP}"
  echo "   Take note of this value in the unlikely event it was"
  echo "   already set by another application on your system"
  echo "   in which case you may need to restore this value later"
  echo ''
  for i in {1..6} ; do echo '******************' ; done
fi
LAB_WRAP="--wrap 0"
echo test | base64 ${LAB_WRAP} 1>/dev/null 2>&1 || unset LAB_WRAP 
if [[ -z "${LAB_WRAP}" ]]; then
  echo "No wrap argument to base64 will be used in the labs"
else  
  echo "wrap argument ${LAB_WRAP} to base64 will be used in the labs"
fi
```

Click the **Next** link in the lower right to see directions to set a few more environment variables.

