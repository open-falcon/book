<!-- toc -->

# Windows System Monitor

In [Data Collection](../philosophy/data-collect.md), we introduced the common data sources. As a monitor framework, Open-Falcon can collect the monitor index data of any system as long as they are converted to the standard format of Open-Falcon.

The running data collection of Windows system, like memory usage, Cpu usage, disk usage, data traffic, and etc., can be done every minute through a Python script in scheduled tasks of Windows.s

The following monitor programs can collect the monitor metrics of machiens with Windows system:

- [windows_collect](https://github.com/freedomkk-qfeng/falcon-scripts/tree/master/windows_collect)：python script
- [windows-agent](https://github.com/LeonZYang/agent)： Agent realized in Go language
- [Windows-Agent](https://github.com/AutohomeRadar/Windows-Agent)：Agent executes as a Windows service open-sourced by AutoHome realized in Python
- [windows-agent](https://github.com/freedomkk-qfeng/windows-agent)：another Windows-Agent realized in Go language supporting port monitor, process monitor and service running in the background

