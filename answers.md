# Getting Started with Datadog on Ubuntu 12.04 and MySQL

## Running Ubuntu 12.04 on Vagrant (Level 0)

* This guide assumes that a working copy of Vagrant is installed
* Please refer to this [link](https://www.vagrantup.com/docs/installation/) for details on how to install Vagrant.
* Starting the VM:
```
$ vagrant init hashicorp/precise64
$ vagrant up
```
* Accessing the VM:
```
$ vagrant ssh
```

## Collecting Data from Ubuntu 12.04 (Level 1)
The fist step is to sign-up an account with [Datadog](https://www.datadoghq.com/).

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Signup.png)

The next step is to provide some optional information about your stack and organization.

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Stack.png)


### Installing the Datadog Agent on Ubuntu 12.04
The next screen will ask you to setup a **Datadog Agent**.

* What is a Datadog Agent?

    A Datadog Agent is a tool that collects data or metrics from the host OS and sends it to the server (Datadog) which can then be interpreted for monitoring.

> Please refer to this [link](http://docs.datadoghq.com/guides/basic_agent_usage/) for more information about the Datadog Agent.

* Select the OS where you want to install the Agent and run the command shown in the screen:

```
$ DD_API_KEY=<YOURKEY> bash -c "$(curl -L https://raw.githubusercontent.com/DataDog/dd-agent/master/packaging/datadog-agent/source/install_agent.sh)"
```
> Tip: the value of `DD_API_KEY` associates the installed Agent to your Datadog account.

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level1-2.png)

* If the installation is successful, the **Finish** button will be enabled in the screen.

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level1-0.png)

* You can also add **tags** to the Datadog Agent by updating the file `/etc/dd-agent/datadog.conf`

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level1-3.png)

For example:
```
# Set the host's tags (optional)
# tags: mytag, env:prod, role:database
tags: stack:vagrant, env:test, role:server
```

> Please refer to this [link](http://docs.datadoghq.com/guides/tagging/) for a detailed guide to tagging.

* You can see the tags you've set in the **Host Map**.

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level1-4.png)

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level1-5.png)

### Creating a MySQL Integration
* Install MySQL on Ubuntu 12.04
```
$ sudo apt-get update
$ sudo apt-get install mysql-server
```
* Configure MySQL for Datadog Integration
```
$ sudo mysql -e "CREATE USER 'datadog'@'localhost' IDENTIFIED BY '<UNIQUEPASSWORD>';"
$ sudo mysql -e "GRANT REPLICATION CLIENT ON *.* TO 'datadog'@'localhost' WITH MAX_USER_CONNECTIONS 5;"
```
* To collect full metrics, grant the following:
```
$ sudo mysql -e "GRANT PROCESS ON *.* TO 'datadog'@'localhost';"
$ sudo mysql -e "GRANT SELECT ON performance_schema.* TO 'datadog'@'localhost';"
```
* Configure the Datadog Agent to collect MySQL Metrics
  * Create a file called `/etc/dd-agent/conf.d/mysql.yaml`:
```yaml
init_config:

instances:
  - server: localhost
    user: datadog
    pass: <UNIQUEPASSWORD>
    tags:
        - env:test
        - stack:vagrant
    options:
        replication: 0
        galera_cluster: 1
```
* Verify the MySQL Integration
```
$ sudo /etc/init.d/datadog-agent info

Checks
======

  [...]

  mysql
  -----
      - instance #0 [OK]
      - Collected 8 metrics & 0 events
```
* Restart the Datadog Agent for the changes to take effect
```
$ sudo /etc/init.d/datadog-agent restart
```
> Please refer to this [link](http://docs.datadoghq.com/integrations/mysql/) for a detailed instruction on how to setup a MySQL integration
* Installing the Datadog MySQL Integration (from UI)
    * Go to Integrations page

    ![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level1-6.png)

    * All available integrations are listed. Select MySQL.
    ![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level1-6.1.png)

    * Follow configuration instructions then install

    ![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level1-9.png)

    [ conf.d/mysql.yaml ]

    ![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level1-9.1.png)

    * After installing, you will now see the database integration in the Integration Dashboards

    ![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level1-7.png)

    MySQL Dashboard:

    ![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level1-8.png)


### Creating a custom Agent Check
* What is an Agent Check?

    Agent Check is a way tp collect metrics from a data source. Click [here](http://docs.datadoghq.com/guides/agent_checks/) to know more about Agent Checks.

* Writing a custom Agent Check

  Create a configuration in conf.d. Each check has a configuration and these are written using YAML.

  ```yaml
  init_config:

  instances:
      [{}]
  ```
![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level1-9.2.png)
  min_collection_interval is set to 20 so that metric can be collected as often as 20 seconds.


  Create a check in checks.d. Note that checks and configuration names should match.

  Below is a custom check that samples a random value


  ```python
  import random
  from checks import AgentCheck
  class RandomCheck(AgentCheck):
      def check(self, instance):
          self.gauge('test.support.random', random.random())
  ```

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level1-10.png)



* Testing the custom Agent Check
```
$ sudo -u dd-agent dd-agent check random
```
* Restart the Datadog Agent for the changes to take effect
```
$ sudo /etc/init.d/datadog-agent restart
```
* View Metrics in **Metrics Explorer**

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level1-11.png)

## Visualizing Data (Level 2)
* Clone database integration dashboard

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level2-1.png)

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level2-2.png)

* Add additional database metrics. Click "Add Graphs" on the right side of your integration name.

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level2-2.1.png)

* Select and add widget on to the board

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level2-2.2.png)

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level2-3.png)

* Add test.support.random metric from the Custom Agent check into the dashboard

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level2-4.png)

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level2-5.png)

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level2-6.png)

* Snapshot of test.support.random showing that it is going above 0.90

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level2-7.png)


### What is the difference between a timeboard and a screenboard?

Timeboards are scoped at the same time while screenboards can have widgets tha each have a different timeframe.

Timeboards are used for correlations, while screenboards are for sharing data.

Sample of Timeboard:

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Timeboard.png)

Sample of Screenboard:

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Screenboard.png)

 Reference: https://help.datadoghq.com/hc/en-us/articles/204580349-What-is-the-difference-between-a-ScreenBoard-and-a-TimeBoard-

## Alerting on Data (Level 3)

* Setup a monitor that alerts when it goes aabove 0.90 at least once during the last 5 mins.

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level3-1.png)

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level3-2.png)

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level3-3.png)

Email was sent when metric is beyond the threshold
![](https://dl.dropboxusercontent.com/u/10874665/datadog/Monitor Alert.png)

* Schedule Downtime for this monitor that silences it from 7pm to 9am daily

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Level3-4.png)

Email sent on downtime

![](https://dl.dropboxusercontent.com/u/10874665/datadog/Downtime Email.png)
