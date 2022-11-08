# Persistence through GPOs

**A quick note here. These techniques are incredibly invasive and hard to remove. Even if you have sign-off on your red team exercise to perform these techniques, you must take the utmost caution when performing these techniques. In real-world scenarios, the exploitation of most of these techniques would result in a full domain rebuild. Make sure you fully understand the consequences of using these techniques and only perform them if you have prior approval on your assessment, and they are deemed necessary. In most cases, a red team exercise would be dechained at this point instead of using these techniques. Meaning you would most likely not perform these persistence techniques but rather simulate them.**

Group Policy Management in AD provides a central mechanism to manage the local policy configuration of all 
domain-joined machines. This includes configuration such as membership to restricted groups, firewall and AV 
configuration, and which scripts should be executed upon startup. While this is an excellent tool for management, 
it can be targeted by attackers to deploy persistence across the entire estate. What is even worse is that the 
attacker can often hide the GPO in such a way that it becomes almost impossible to remove it.

Common GPO Persistence Techniques

* Restricted Group Membership
* Logon Script Deployment
* Firewall Tampering
* Anti-Virus Tampering

## Resources

* [Sneaky Active Directory Persistence #17: Group Policy](https://adsecurity.org/?p=2716)





