# Broken Authentication Part 1

Category: Web
Difficulty: Easy
Status: TODO
Tags: web

***TABLE OF CONTENTS:***

---

# Goal

The goal of this challenge is to find a way to bypass the authentication to retrieve the flag.

# Description

From the landing page on [http://MACHINE_IP:5000,](http://MACHINE_IP:5000,) go to Broken Authentication under Track: Vulnerable Startup ([http://MACHINE_IP:5000/challenge1/](http://machine_ip:5000/challenge1/)).

# Task

Bypass the login and retrieve the flag.

# Flag

THM{f35f47dcd9d596f0d3860d14cd4c68ec}

---

---

The code `' OR ''=''` effectively evaluates to true, allowing the attacker to bypass any authentication barrier.