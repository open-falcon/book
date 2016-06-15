##Fe

It is the UIC of Go version and also a unified Web interface. Because of many monitoring components, it is troublesome to remember IP and port for access. Fe is like hao123 for monitoring.

##Design Intention

The level of difficulty of software deployment and distribution directly affects the popularity of software. Some people in the community responded that the UIC of Java version was difficult to install. Therefore, we provide a Go version. You can maintain your personal contact information and the correspondence between individual and group in Fe. You can also generate multiple shortcut links on the page through the shortcut configured in the configuration file.

##Differences from UIC

In addition to the simple navigation that the Fe module provides, the biggest difference is that the way the password is stored has changed. Therefore, for Java UIC users to migrate to Fe, salt in the Fe module needs to be modified to an empty string so that Fe can use the same database with the original Java UIC. However, it is not secure to set salt to an empty string. It is recommended to set salt to a random string. Then register a new user through Fe and reset the passwords of all users in the database to the password of the new user. Send a notice to instruct each registered user to log in again and modify their password.

##Source Code Installation

```
cd $GOPATH/src/github.com/open-falcon/fe
go get ./...
./control build
./control pack
```
A tar.gz package will be packed at the last step. We can deploy with this package.


##Deployment Instruction

As a front-end module, Fe is stateless and can scale out. Deploy two computers at least to ensure availability. Set up a load-balancing device. Either nginx or lvs can work. Finally, apply for a domain name for it.

##Configuration Instruction

The name of the configuration file must be cfg.json. We can change the configuration file based on cfg.example.json.

```
{
    "log": "debug",
    "company": "MI", # any port that is not duplicate with the existing ports.
    "http": {
        "enabled": true,
        "listen": "0.0.0.0:1234" # You can open a random port which can not repeat now. To be consistent with old version,you can use 8080.
    },
    "cache": {
        "enabled": true,
        "redis": "127.0.0.1:6379", # This Redis is different from the Redis used by Judge and Alarm and is only used as cache. It is recommended to set up a separate Redis instance that is used as cache.
        "idle": 10,
        "max": 1000,
        "timeout": {
            "conn": 10000,
            "read": 5000,
            "write": 5000
        }
    },
    "salt": "0i923fejfd3", # a random string
    "canRegister": true,
    "ldap": {
        "enabled": false,
        "addr": "ldap.example.com:389",
        "baseDN": "dc=example,dc=com",
        "bindDN": "cn=mananger,dc=example,dc=com", 
        "bindPasswd": "12345678",
        "userField": "uid",
        "attributes": ["sn","mail","telephoneNumber"] 
    },
    "uic": {
        "addr": "root:password@tcp(127.0.0.1:3306)/uic?charset=utf8&loc=Asia%2FChongqing", # The schema database is under the scripts directory.
        "idle": 10,
        "max": 100
    },
    "shortcut": {
        "falconPortal": "http://11.11.11.11:5050/", # portal address that the browser can access
        "falconDashboard": "http://11.11.11.11:7070/", # Dashboard address that the browser can access
        "falconAlarm": "http://11.11.11.11:9912/" # http address of Alarm that the browser can access
    }
}
```
##Process Management
We provide a control script to complete common operations.

```
./control start Start the process
./control stop Stop the process
./control restart Restart the process
./control status View process status
./control tail View var/app.log using tail -f
```
##Set the password of the root account
Registered users in this project have different roles. Currently, there are three roles: common users, administrators, and the root account. After the system starts up, first you need to set the password for the root account. Use your browser to visit http://fe.example.com/root?password=abc (it is assumed here that your project access address is fe.example.com and you can also use the IP address). Thus, you set the password for the root account to abc. Common users are allowed to register.


##ladp

Now Fe supports user authentication via ldap. Without opening an account in advance, Fe will automatically insert new users approved by ldap into database within Fe.

Configuration note
```
        "addr": "ldap.example.com:389",
        # ldap address and port

        "baseDN": "dc=example,dc=com",
        # ldap’s baseDN, querying user from this path when ldap authentication occurs

        "bindDN": "cn=mananger,dc=example,dc=com", 
        # the account you used to connect ldap have at least read-only query access.
        # note that it should be account’s complete dn value. As for AD, directy fill in account’s userPrincipalName (xxx@example.com）.
        # if your ldap is allowed for anonymous query, just need filling in "" value

        "bindPasswd": "12345678", 
        # if your ldap is allowed for anonymous query, just need filling in "" value

        "userField": "uid", 
        # attribute for authentication (user name you entered), usually as uid or sAMAccountName(AD).
        # also used attributes like mail , so user name as authentication is mail (provided that there is the attribute in ldap)

        "attributes": ["sn","mail","telephoneNumber"] 
        # array sequence priority, attribute name is as follows: name, mail, telephone number.
        # recommend that modify according to your own ldap.
        # when user is logging in ldap, fe will query attributes of new user according to these attribute names, and insert them into database within fe.
    },
    ```


##Supplementary Note
Here, we first installed the Fe module while the Portal, Dashboard, and Alarm modules have not been installed. Therefore, you may not be sure about how to configure shortcut. Don't worry. Keep the default value and then return to modify the Fe configuration after deploying the Portal, Dashboard, and Alarm modules.

##Video Course

We recorded a video for the Fe module to provide interpretation at the source-code level:http://www.jikexueyuan.com/course/1780.html



