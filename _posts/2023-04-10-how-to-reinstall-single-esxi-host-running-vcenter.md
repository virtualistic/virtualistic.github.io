---
title: "How to reinstall a Single ESXi Host Running Your vCenter"
date: 2023-04-10T16:47:34+02:00
categories: [homelab,vmware]
tags: [esxi 7, vcenter, reinstall, powercli]
---

Some quick notes on how I reinstalled my homeserver from 6.7 to 7 without losing any VM configuration

<!--more-->

## The Challenge

I was still running ESXi 6.7 from a usb-stick and I knew that going forward VMWare is no longer going to support this type of setup.
So how am I going to reinstall the host from scratch that also hosts my vCenter without losing all my VM configurations ?
Recipe:
- Shutdown all VMs except vCenter
- Make note if you have any VMs with manual mac addresses. If so collect that information
- Run the following one-liner from Powercli to save the location info of the VM's to a csv file
- ```powershell
  Get-VM | Select Name, @{N="VMX";E={$_.Extensiondata.Summary.Config.VmPathName}} | Export-CSV -Path "c:\temp\vm-info.csv" -NoTypeInformation
  ```
- IMPORTANT: in vCenter disconnect the host (right-click, disconnect)
- Remove any PCIE passthrough & zwave USB devices from the VMs
- Shutdown vCenter
- Shutdown the host
- Remove the ESXi 6.7 USB thumbdrive
- Install ESXi 7 to a local SSD using the same hostname & IP
- After installation run this script to re-register all VMs again
- ```powershell
  connect-viserver "esxi7-host.local"
  $csv = Import-Csv -Path 'C:\temp1\vm-info.csv'
  foreach ($row in $csv){
    write-host "registering" $row.name -ForegroundColor Green
    $vm = $row.name
    $vmxpath = $row.VMX
    Write-Host "VM name is:" $vm
    Write-Host "vmx path name is:" $vmxpath
    New-VM -Name $vm -VMFilePath "$vmxpath"
    Write-Host ""
  }
  ```
- select devices for passthrough (a HBA in my case) and reboot after selecting
- attach the PCIe device back to the VM
- attach attached usb devices back to its VM
- Start vCenter, login, select the host and click: "Connect". Accept the new certificate
Done!
