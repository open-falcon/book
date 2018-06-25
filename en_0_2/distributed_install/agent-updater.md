<!-- toc -->

# Agent-updater

Falcon-agent needs to be deployed in every machine. If the quantity of company's machine is relatively small, it does not matter that you install falcon-agent manually using tools like pssh, ansible and fabric. But when the quantity increases, it will become a nightmare that you finish all the installation, update and rolling back manually.

I personally developed a tool called Agent-updater for Falcon-agent management. Agent-updater also has a agent called ops-updater, which can be considered as a super agent that manage the agent of other agents. Ops-updater is recommended during installing. Usually, it does not require an update.

For more information, please visit: https://github.com/open-falcon/ops-updater 

If you want to learn how to write a full project using Go language, you can also study agent-updater. I even recorded a video tutorial to show you how it is developed. The links are down below: 

- http://www.jikexueyuan.com/course/1336.html
- http://www.jikexueyuan.com/course/1357.html
- http://www.jikexueyuan.com/course/1462.html
- http://www.jikexueyuan.com/course/1490.html

