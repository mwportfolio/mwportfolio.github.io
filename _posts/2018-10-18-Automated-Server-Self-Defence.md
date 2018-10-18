---
layout: post
title: "Automated Server Self-Defence"
date: 2018-10-18
---

This post describes a solution to automate a defensive firewall configuration when faced with many repeated attack attempts on a server.

## Why? 

&nbsp;

I developed this solution because when I was checking or sending my email on my phone, the application often timed out, was unresponsive or experienced very slow performance.

## How?

&nbsp;

![Solution Overview](https://www.draw.io/?lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=Untitled%20Diagram.xml#R7ZpRb6M4EIB%2FTR6JwCYJeQxp2V1pV6rUk%2B4xcsEBqwYj4zTkfv3ZYBLANA27adXdTas2eAy2mW%2FGnrEzgeu0%2FMJRnvxgEaYTYMecRBN4NwHAkX9SEJW66IK5lhxqyWK2rAXqmUfyH9Y32lq6IxEuOm0JxqggeVcYsizDoejIEOds371ty2h3YDmKsSF4DBE1pf%2BSSCS11AOLk%2FwrJnHS9OzM9es8ofA55myX6f4mAG6rn7o6RU1b%2BkWLBEVs3xLB%2Bwlcc8ZEfZWWa0yVbhvN6ucmIHjlhuPQOc7Ehc%2FA%2BoYXRHe4GXc1OnFoNIKzaKUUK0sZy6TQT0RKZcmRl2aXehQ46uhZ9%2F4FsxQLflCwNKj4KFKPJS3dzvTo9ycOjYhjigR5GejhgRE5klPzBdvxEGupelSbIbR1U9pQh3TVb0sgHmNhtgW8t9qSF62XP4kqGsNk3BsZXW4mjCuQMdv6CTKzG5mm5ev5jNnWT5BZGGS%2BylehB4OPwKXoQikEZ894zSjjJ2hbQmlPhCiJM1kMJTws5f4L5oLI9WOlK1ISRaobf58QgR9zFKo%2B93K1lLJqfcBqtLZqnmWiaV4uF371q%2BWPerD2q9aiOsblL9oLMO3Ftc9PlMOgzlDxBvzFn6U3KNeDAuHgvHYGynIIimPfqFyRiuuNdRXHMbB8R4WaTxM5kRXyk7K4%2BD0Yraufc%2BvdVbBAE8uyi8XxZh0szmI2FgswsKzimOOiUOspsL893Ji8wcSxz0OB9tgZzDFzl3842m5JqPI%2BztIbmCs4izsb7SwXxMcy%2Bc3VJUmrdLwNo69TwfKW9Dt6wvSBFUQQpmqfmBCSNPSpqvCPOXhLtToLh37V2arI620DBQE1hS0pFRlfj%2BcuEULtN6zUS4MgjDI4JSHLtkQS5NNQ2VYQIYHkh5JLIwsKFhJErRRHBFmg2hYIVBKnKzZVxaZTkHM520QsfMbccoA3zbP4w%2FHDLn7ggl60Nxr%2F%2FA%2FED97Ez8pDjDN5MXfLucKeEgnhkGNVqaYcq7Tyg0hY9gGYgYkZ9DDD7uzblC%2FHbOZaBuYBvJdyqoxDqimg5EkhoCTfIC4DoSDDYs%2F4M1FKDALC8R5RupHuU36ICzXZzDkX6urW9Ua7kBmcr7m097%2FAjTK8tw4YcUsGV4zu1PiKyqnkv5bIst3jjPmJ1tKeJcB5L0tzx1pCk3NcMpmGaqfD5xIeFq%2BEIC0DOUUjdUkgbQvW0n531R03f1q6Az0vMnYBx%2BZSA3vbd7s0%2F8TxoD4NcbxeYuucSYDfGZTJyetxcnrJ1eicd2Cn%2B1sk34ZsP%2FO%2B3Z%2FACg6fU5xjZcb2d5zdfOp9ObnzsZyguY9kMBqIFn4lLhiOOI6nnIpHhIrkCKc2iOaMFdT3VgtZWsbqbHlKWLGYpqQIp7zWXs%2BIZO%2Br5dq9919BPYTWMK8PjxH669ysxxqOZm1uTt1Y%2FxaswVtrpQp6j18IqM%2FDTt%2B6gPf%2FAw%3D%3D)

&nbsp;

#### Dump

I investigated the logs for the mail server and identified potential issues that could impact performance.

&nbsp;

Then I scheduled hourly cron job to dump the last hours worth of logs from the mail server to a file, overwriting the file every hour with the latest logs.

&nbsp;

![Docker logo](https://png.icons8.com/color/50/000000/docker.png)

~~~ bash
docker logs --since 1h <servername> > recentlogs.txt

~~~

&nbsp;

#### Identify

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

#### Drop

&nbsp;

I developed a bash script which inserted firewall with rules to DROP traffic from the aggressive IP addresses saved in the CSV file. The bash script was scheduled to run every hour staggered 5 mins after the Python script cron job.

&nbsp;

![Bash logo](https://github.com/odb/official-bash-logo/raw/master/assets/Logos/Icons/PNG/64x64.png)

~~~ bash
for f in `cat $output_filename`; do iptables -I INPUT -p tcp -s $f -j DROP; done 

~~~

&nbsp;

## Result?

&nbsp;

The end result was that acceptable performance of the mail server and client application was restored and an automated self-defence solution was established to prevent further attack attempts.
