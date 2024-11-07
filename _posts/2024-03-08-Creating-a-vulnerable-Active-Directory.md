---
layout: post
category: pentesting
---
today we will be creating an vulnerable Active Directory. It will be helping us to understand how this works, how the attacks works, and more. 

first things first:

# What is Active Directory?

a directory service, such as ***Active Directory Domain Services (AD DS)***, provides the methods for storing directory data and making this data available to network users and administrators. For example, AD DS stores information about user accounts, such as names, passwords, phone numbers, and so on, and enables other authorized users on the same network to access this information.

![AD](https://raw.githubusercontent.com/notluken/notluken.github.io/master/_screenshots/ad.jpg){:.ioda}

Active Directory is like a giant address book for a computer network. it organizes information about all the users, computers, and other resources on the network in a structured way. think of it as a filing cabinet where everything has its place.

it also has a set of rules called ***the schema***, which define what kinds of things can be stored in the directory and how they're organized. this helps keep everything tidy and organized.

one cool feature is the ***global catalog***, which is like a super index that contains information about everything in the directory. So no matter where you are on the network, you can find what you're looking for.

another handy feature is the ability to search for things quickly using queries and indexes. this makes it easy for users and applications to find what they need.

and finally, there's a replication service that makes sure all the information in the directory stays up to date across the network. so if something changes, like a user's password or a new computer is added, it's quickly spread to all the other parts of the network.

this is a simple resume of what Active Directory is. if you're new into this do not worry. it's hard at the beginning but it keeps being easier while you're practicing.

see the [Learn Microsoft page of Active Directory](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview) for more information.

# What are we going to need

we're going to need some things in order to create our Active Directory Lab:
* VMware/Virtualbox
* Kali/Parrot OS
* Windows Server ISO
* Windows Enterprise ISO

# Firsts Steps



