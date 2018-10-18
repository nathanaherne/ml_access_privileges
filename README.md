As a co-founder of Rulerr.com and being passionate about Machine Learning, I felt that making a machine learning algorithm available on Github may help those organizations who are unable to use an external party to look for anomalies in their Access Privileges.

**Current Status of this document** -> very much a dump of my thoughts.

# Introduction

This document outlines the design of a machine learning algorithm with the end goal of helping an organization manage the access privileges for their staff or customers.

Access Privileges tell you much more about an organization than you would expect, this extra information is especially useful for Managers and Executives. Spoiler Alert, Access Privileges tells you:

1. How an organization is structured hierarchically based on the sensitivity of information each person has access to.
	a. It often comes as a shock to most Managers and Executives, that they are very low down on the "totem pole" (so to speak) in comparison to other employees when it comes to access to information.
	b. On a positive note, the data gives Managers and Executives visibility on who can effect change within an organization because they can actively see who knows the organization the best.
2. How risky each person is in the organization
	a. This helps organizations target specific users for training on best security practices.

## Input Data

To create an algorithm for access privileges, we must first consider what data would be most readily available and build upon that data over time:

1. Information about each entity (for example a person) in the organization
2. Access privileges for each piece of software/hardware/building etc

The quality of the information provided to the model will increase the available information it can provide the organization.

Many organizations have hundreds, if not thousands of separate access privilege systems that are not integrated or validated against other internal systems. Considering this, the model should provide high value information using the least amount of information possible.

The model(s) should be able to consider more complex information (to the point of ideal) as that information becomes available.

### Information required for the model

**Minimum Possible**

For the model to create basic recommendations for access privileges, it needs some data about access privileges for the organization. The minimum amount of data that the model needs to provide useful information about access privileges is the access privileges for each user for a single piece of software/hardware/building etc.

**Entity information**

The model needs a way to differentiate between access privileges for one entity and another. In the **Minimum** section, the model assumed that each username/password was a different entity but that may not always be the case. If we were to add access privileges for another piece of software/hardware/building, the model will still be considering each username/password as a separate entity, that no two entities have access to each piece of software/hardware/building (and this is highly unlikely).

By providing entity information connected to access privileges, the model is now able to infer much more about your organizations access privileges.

The entity information might be as follows:

1. FirstName and LastName
2. Department(s)
3. Job Role (they may have multiple or none in the case of a computer system)
4. Sex (male/female)
5. their work address
6. their direct manager
7. email address(s)
8. Phone details
9. Their home address (or at least the city/suburb/zip/country)
10. employment status

BTW: I call it "entity information" instead of "people information" because many organizations have many computer systems, machine learning algorithms and computer programs that can access many different systems within the organization. 

When collating entity information, you can choose to only use real people and discard any accounts that cannot be attached to actual people. Doing this would produce an incomplete model but it may make things simpler to start with.

**Pre-processing**

The easiest way to get data on the people within your organization would be to get access to Human Resources information about each person in your organization. Another place to get information would be from the Payroll department as they are usually better at knowing who is employed and who isn't.

Entity information and Access Privilege information are almost always separate pieces of data, so we need some way to match them quickly. Most access privilege information has a username component and that can often be the entities name, email address or phone number. The pre-processing script would look for matches between this information.

It makes sense that each account in a piece of software/building/hardware etc is attached to an entity or you do not know who has access to what. If your organization does not match accounts to entities, over time you will want to match all usernames with entities within the organization.

# Considering the data

To create a good model, we need to understand what the model might be able to infer from the data you can feed it. Remember that computers cannot understand context, so context needs to be inferred from the data itself without outside influence.

## Access Privilege data

To help us understand what access privilege data might infer to the model, I will be using an analogy because I find it easier to understand access privileges when it is attached to a simple real world example.

I will be using Building Access (a house in this case) to help create a model of what access privilege data can infer.

**Having access means you are able to use it**
If you have a key to a house, you can access the house.

*Terms*
This key would be called the houseKey.

**Access is generally hierarchical**
If all the rooms in the house have a door with a key - just because you have the key to the house, does not mean you can access all the rooms.

In the real world, doors that protect more sensitive objects are usually more physically secure - think your front door vs a bank vault.

*Terms*
This key would be called the doorKey.

*Use in our model*
In a world where software is used to grant access to both real and metaphorical doors, there is little concept of sensitivity - its just as easy to give the key out for the bank vault as it is to do so for any other door. To create the concept of sensitivity, we must create a model which infers sensitivity from other information.

**The key provides no information**

If you look at the keys on your keyring, it is very difficult to know which one gives you access to more sensitive objects. Considering it another way, if you found a key on the street, you wouldn't know what was behind the door.

*Use in the model*
Keys themselves hold no intrinsic value.

**The number of keys could infer something**

Having many keys could infer some level of importance within an organization. In the real world, the number of keys could not denote importance alone, think of a janitor that needs the keys to all doors.

*Use in the model*
The number of keys held by an entity could denote some level of importance.

**The door provides no information**

A door does not tell you the how sensitive the objects behind it are. Sure, there is a physical different between a bank vault and a front door but I've seen many bank vaults used to store objects with no sensitivity at all.

*Use in the model*
Just because there is a door does not tell you anything.

**Having the key does not mean you know what is behind the door**

If you have never gone through the door or had the sensitivity of what is behind the door explained, you do not know the sensitivity of the objects the door secures. Even if you have gone through the door and looked through all the objects, you may not know its sensitivity. A good way to understand this is being able to value art - I would have a hard time know a valuable painting from a non valuable painting.

*Use in the model*

We can infer a certain amount of knowledge from access:

1. People who access the door more often than others are more likely to understand the sensitivity.
2. People who access the door for longer than others are more likely to understand the sensitivity.

**Having access to a door that few people can access, infers importance**
If 


# Training the Model

To train the model we need to consider how the data can be used by the model to find anomalies in your access privilege information and in time be able to help manage your organizations access privileges.

To train the model, the model needs to know what is correct and what is incorrect.

What is correct:

1. People within the same department and job role should *probably* have similar Access Privileges.
	a. The above statement should be considered with the caveat that people who have moved positions may still have privileges from those positions.
2. All accounts within a piece of software, should be attached to an entity.
4. People who are no longer employed by your organization should not have access to any systems.

# Training

Most organizations would have some incorrect permissions. Teaching the model based on existing permissions and treating them as correct would not produce the desirable outcome. Before training the model, we would need to normalise the input data to remove permissions that do not fit these particular rules. To train the model, we would need to attach accounts with people. Those accounts that were not attached to people would be flagged as incorrect.

An alternative to this method would be to train loosely to the dataset and not exclude any data. This would then allow us to flag any outlying data points from the model.
