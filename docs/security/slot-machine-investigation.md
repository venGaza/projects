---
layout: default
title: Slot Machine Investigation
parent: Security
nav_order: 2
---

# Slot Machine Investigation
{: .fs-9 .no_toc}

Objective 4 from the 2021 Holiday Hack Challenge from SANS Institute 
{: .fs-6 .fw-300 }

[Solution](#solution){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 } [View CTF](https://www.sans.org/mlp/holiday-hack-challenge/){: .btn .fs-5 .mb-4 .mb-md-0 }

---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Challenge

Test the security of Jack Frost's slot machines. What does the Jack Frost Tower casino security team threaten to do when your coin total exceeds 1000? Submit the string in the server data.response element. Talk to Noel Boetie outside Santa's Castle for help.

## Solution

Jack Frost is at it again with his proprietary slot machines, but suspects there may be a vulnerability in his web application with so many elves walking away winners! To start, we are redirected to the following link: https://slots.jackfrosttower.com. 

We start our enumeration by familiarizing ourselves with how the application works. The player starts with 100 credits and has the choice to select both a "bet size" and a "bet level". As they are increased, the amount of the bet is increased for the spin. When the player spins the machine, 3 rows of 5 icons spin and the player wins if they match a preselected pattern outlined in the paytable. Simple enough! After gambling away my life savings on 16 games and walking away with nothing, I decided to drown away my sorrows with a tall glass of eggnog. How were these other trolls getting rich but not me?! 

For this challenge, we can use the Firefox browser web developer tools to analyze the network traffic between our slot machine (the client) and the frost tower servers. When we press the spin button, our browser submits a POST request to Jack's server. We can inspect this request and see all the headers and cookies, but the most important part is the body. 

```CodeBlock
betamount=1&numline=20&cpl=0.1

```

We see there are 3 parameters being passed to the server: betamount, numline, cpl. It may be possible to adjust the values of these parameters and improve our so-called luck! We need to figure out what these parameters actually correspond to. To accomplish this, we will spin the slot machine a few more times with different values for "bet size" and "bet level" and then re-inspect the post requests to see how the POST body parameters change.

```CodeBlock
betamount = aligns to the bet level
Numline = remains constant through each spin but probably corresponds to the number of ways to win 
cpl = aligns to the bet size

```

Now we know what these parameters do, but how do they affect the logic of the server behavior? We will now look at the response to our POST request to search for clues on how to proceed.

```CodeBlock
data	Object { credit: 98, jackpot: 0, free_spin: 0, ... }
credit	98
jackpot	0
free_spin	0
free_num	0
scaler	0
num_line	20
bet_amount  1

```

The response shows that a JSON object is returned with various values the server keeps track of for each spin. It reveals very little about the actual logic of how a deduction or addition occurs, so we will need to experiment with the parameters to begin to deduce the inner workings of this server.

One approach for experimenting with the parameters is to borrow a framework from the software development world known as ZOMBIES. This acronym stands for:


```CodeBlock
Zero, One, Many, Boundaries, Interface, Exceptions, Simple

```

This framework provides a methodical approach for software testers to create test cases to identify bugs in an application. In our case, we can see how the server responds when a parameter is presented with edge case values it may not be expecting.

For example, we know betamount can be zero but cannot be an extreme value around 100 or else the server will respond with an error. It also cannot be a negative value or an error will occur. We know the boundaries are 0 to ~20. Decreasing this parameter to zero does stop us from losing credits, but it also stops us from gaining credits when we win.

We can do the same process with the other two parameters. However, we find something strange happens with the server response when we input negative values. Instead of credits being deducted on a loss, they are instead added... holy Rudolph!!!

We have discovered an input validation vulnerability on Jack Frost's server. Even though we do not know the formula being used by the server to calculate loss/win amounts, we can deduce that the server is subtracting an amount that is multiplied by either numlines or cpl. The server is trying to subtract a negative number on losses which makes the amount positive instead. After adjusting the bet amounts and resending the request a few more times, Jack Frost has finally had enough as noted by the flag value in the server response:


```CodeBlock
I'm going to have some bouncer trolls bounce you right out of this casino!

```