# Nagios check for SSD Wear Level

This is a small script to check SSD Wear Level Indicators, specifically SMART ID 177 and 233, or the wear-leveling attribute in nvme-cli.

This script tries to find SSDs itself by checking for RAID controller cards we use within our organization. As of this moment that is 3Ware and LSI/AVAGO cards.

It also supports IDE/SATA/NVMe drives.

## Example

IDE/SATA:

    # check_ssd
    SSD OK: (auto) Drive /dev/sda WLC/MWI 100. Drive /dev/sdc WLC/MWI 100.

NVMe:

    # check_ssd
    SSD OK: (auto) Drive /dev/sda on 0 WLC/MWI 99. Drive /dev/sdc on 0 WLC/MWI 99. Drive /dev/nvme0n1 on 0 WLC/MWI 100. Drive /dev/nvme1n1 on 0 WLC/MWI 100.

3Ware 9750 Controller:

    # check_ssd
    SSD OK: (3ware) Drive 2/0 WLC/MWI 99. Drive 3/0 WLC/MWI 99.

LSI 9341 Controller:

    # check_ssd
    SSD OK: (megaraid) Drive 4/1 WLC/MWI 83. Drive 5/1 WLC/MWI 83.

## Usage

The script supports the following parameters:


    -c=,  --card=              Instead of autodetecting using lspci, set the card type. We accept "lsi", "3ware" and "auto" for now. Auto is autodetect
    -d=,  --device=            The blockdevice we should use for calling smartmontools. Can be any blockdevice in /dev, but is driver specific
    -b=,  --brand=             The brand of SSD to search for. We accept "samsung" and "intel"
    
    -d,   --debug              Enable debug output
    -t,   --test               Only test if there are SSDs present in the system. exits with 0 when found, 1 if not
    -h,   --help               Show what you are reading now!

## Requirements

The script uses the following tools:

- smartctl for retrieving SMART data
- storcli for managing LSI controllers
- tw_cli for managing 3Ware controllers
- nvme-cli for managing nvme disks
- bc for calculations
- awk for string manipulation
- head/tail for output filtering
- sed for string manipulation
- tr for removing characters
- lsblk to identify SSDs on HBAs (non-RAID)
- lsscsi to lookup blockdevice by controller

## Example

In our case we call the script after the RAID status check completes with exit code zero, and explicitly set the card type within the script. e.g.:

in check_lsi_raid:
        
    sub getSSDstatus {
        system("/usr/bin/check_ssd -t -c=lsi");
        if ($? == 0) {
          # We have SSDs!
          my $command = "/usr/bin/check_ssd -c=lsi";
          my @output = `$command`;
          printf (@output);
          return $? >> 8;
        } else {
          return 0;
        }
    }

    ... 

    if($exitstatus == 0) {
                print "LSIRAID OK (Ctrl #$controller) | STATUS=$exitstatus\n"; 
                my $exitcode = getSSDstatus();
                exit($exitcode);



# ChangeLog:

- Version 1.5: Tiny bugfix
- Version 1.4: Support for NVMe disks.
- Version 1.3: Replace lsblk -S with lsscsi. -S flag on lsblk is not available on Wheezy
- Version 1.2: Correctly identify which blockdevice (/dev/sdX) belongs to which controller
- Version 1.1: Support for multiple LSI cards. Support for multiple 3Ware cards are not yet implemented.

