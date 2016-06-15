# 7.Secondary Development

##The construction of go development environment:
```
cd ~
wget http://dinp.qiniudn.com/go1.4.1.linux-amd64.tar.gz
tar zxf go1.4.1.linux-amd64.tar.gz
mkdir -p workspace/src
echo "" >> .bashrcecho 'export GOROOT=$HOME/go' >> .bashrcecho 'export GOPATH=$HOME/workspace' >> .bashrcecho 'export PATH=$GOROOT/bin:$GOPATH/bin:$PATH' >> .bashrcecho "" >> .bashrc
source .bashrc
```
##clone code
```
cd $GOPATH/src
mkdir github.comcd github.com
git clone --recursive https://github.com/XiaoMi/open-falcon.git
```
##compiling an element(taking agent as an example)
```
cd $GOPATH/src/github.com/open-falcon/agent
go get ./...
./control build
```
##User defined modification filing strategy
Modify open-falcon/graph/rrdtool/rrdtool.go

##picture
 
Compile element graph again, and replace the original binary

Eliminate all the original rrd files（under /home/work/data/6070/ by default)

##Plugin mechanism
1.Find a git to store all the plugins of company

2.Download the repo plugin to the local by calling the /plugin/update interface of agent

3.Deploy which hosts can execute which plugins in portal

4.The naming way of plugin: $step_xx.yy, which needs the execute permission to save to the each directory by classification 

5.Print the collected data to stdout

6.You may modify the agent and download the plugin.tar.gz from a http address at fixed period if you find the git way inconvenient