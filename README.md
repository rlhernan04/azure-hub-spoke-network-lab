# azure-hub-spoke-network-lab

# Overview

This project implements a network segmentation architecture that mirrors how enterprise networks are commonly designed: a central "hub" network containing shared security infrastructure (a firewall), connected to a isolated "spoke" networks representing different application tiers (web and app). Rather than letting the spokes communicate directly, all inter-spoke traffic is forced through the hub's firewall for inspection, and each subnet carries its own network security group as a second layer of access control.

# Why Hub-and-Spoke?

In a flat network, every Virtual Machine can talk to every other Virtual Machine by default. That is fine for a small lab, but it does not reflect how real organizations operate, and it fails the "least privilege" principle that security teams are graded on. A compromised web server in a flat network could pivot straight to a database server with zero friction.

Hub-and-spoke solves this by centralizing the choke point. Instead of trusting each spoke to defend itself, all cross-spoke traffic is forced through one inspection point (the firewall) that can log, allow, or deny traffic based on explicit rules. This also mirrors how real enterprises structure Azure environments: a shared hub VNet holds common services (firewall, VPN gateway, DNS), while spokes hold workload-specific resources.

# Architecture

## IP address planning

vnet-hub -- 10.0.0.0/16 -- AzureFirewallSubnet(10.0.0.0/26) -- Hosts Azure Firewall
vnet-spoke-web -- 10.1.0.0/16 -- snet-web(10.1.1.0/24) -- Web tier
vnet-spoke-app -- 10.2.0.0/16 -- snet-app(10.2.1.0/24) -- App tier

- All resources were grouped under a single resource group (hubspoke-lab) in East US \*

# Build Steps

## 1. Resource group

Created hubspoke-lab to contain everything, keeping the project organized and easy to tear down as a single unit later.

## 2. VNets

Deployed all three VNets (hub,spoke-web,spoke-app) in the same region with non-overlapping address spaces, a prerequisite for peering to work cleanly.

## 3. VNet peering

Peered each spoke to the hub (not to eachother, that is the core design principle: spokes never talk directly). For each peering: - Enabled "allow forwarded traffic" on the spoke side, which is what allows traffic that is been redirected by the firewall to actually reach its destination instead of being silently dropped - Confirmed both peerings showed "Fully Synchronized" and "Connected" status before moving on

## 4. Azure Firewall Deployment

Deployed a Standard SKU Azure Firewall (fw-hub) into the hub's AzureFirewallSubnet, with its own dedicated public IP (10.0.0.4). This became the mandatory inspection point for cross-spoke traffic.

## 5.
