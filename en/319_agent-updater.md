##Agent-updater

It is required to deploy "falcon-agent" for each machine. If there are just a small number of machines in the company, it is OK to install manually with tools such as pssh, ansible and fabric. But if there is a large number of machines in the company, installing, upgrading and rollbacking "falcon-agent" manually can be a nightmare.

A tool named "agent-updater" is developed, which can be used to manage "falcon-agent". "agent-updater" also has an agent: "ops-updater", which can be regarded as a super agent and used to manage the agents of other agents. It is recommended to install ops-updater together when setting the machine up. Usually, ops-upgrader doesn't need upgrades.

Please refer to https://github.com/open-falcon/ops-updater for details.

If you want to learn how to use the Go language to write a complete project, you can study "agent-updater". I have even recorded a Video course to demonstrate how to develop it step by step. Tutorial link:

* http://www.jikexueyuan.com/course/1336.html
* http://www.jikexueyuan.com/course/1357.html
* http://www.jikexueyuan.com/course/1462.html
* http://www.jikexueyuan.com/course/1490.html
