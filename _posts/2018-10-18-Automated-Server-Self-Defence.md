---
layout: post
title: "Automated Server Self-Defence"
date: 2018-10-18
---

This post describes a solution to automate a defensive firewall configuration when faced with many repeated attack attempts on a server.

### Why? 

&nbsp;

I developed this solution because when I was checking or sending my email on my phone, the application often timed out, was unresponsive or experienced very slow performance.

### How?

&nbsp;

In summary, I investigated the logs for the mail server and identified potential issues that could impact performance.

&nbsp;

Then I scheduled hourly cron job to dump the last hours worth of logs from the mail server to a file, overwriting the file every hour with the latest logs.

&nbsp;

![Docker logo](https://png.icons8.com/color/50/000000/docker.png)

~~~ bash
docker logs --since 1h <servername> > recentlogs.txt

~~~

&nbsp;

I developed a Python script using pandas to read the dumped log file and identify IP addresses which had connected to the server more times than a preconfigured limit over an hour, then saved those IP addresses to a CSV file. This Python script was scheduled to run every hour, staggered 5 mins after the log dump cron job.

&nbsp;

![Python logo](https://www.python.org/static/favicon.ico)

~~~ python
# identify aggressive IP addresses hitting server more than predefined limit per hour
df_offenders = df.loc[df['hits'] >= hits_limit, :]

# save offending IP addresses to CSV
df_offenders.to_csv(output_filename)

~~~

&nbsp;

I developed a bash script which inserted firewall with rules to DROP traffic from the aggressive IP addresses saved in the CSV file. The bash script was scheduled to run every hour staggered 5 mins after the Python script cron job.

&nbsp;

![Bash logo](https://github.com/odb/official-bash-logo/raw/master/assets/Logos/Icons/PNG/64x64.png)

~~~ bash
for f in `cat $output_filename`; do iptables -I INPUT -p tcp -s $f -j DROP; done 

~~~

&nbsp;

### Result?

&nbsp;

The end result was that acceptable performance of the mail server and client application was restored and an automated self-defence solution was established to prevent further attack attempts.
