<!-- toc -->

# Environment Preparation

Please refer to [Environment Preparation](../quick_install/prepare.md)
# Changing Custom Archiving Strategy
Change open-falcon/graph/rrdtool/rrdtool.go

![](https://raw.githubusercontent.com/open-falcon/doc/master/img/custom-rra-1.png)
![](https://raw.githubusercontent.com/open-falcon/doc/master/img/custom-rra-2.png)

Recompile Graph module and substitute the existing binary for a newer one

Delete all previous RRD files (saved at "/home/work/data/6070/" by default)

# Plugin Mechanism
1. Find a git that can store all the plugins of our company
2. Pull the Repo pulgin to local system by calling the /plugin/update port of Agent
3. Set which machine can execute which plugin in Portal
4. Plugins are named in form of "$step_xx.yy" and stored with executable permission in each directory of Repo by category
5. Print collected data to Stdout
6. Modify Agent if you find the git method inconvenient, just download zip files "plugin.tar.gz" from certain http address regularly

