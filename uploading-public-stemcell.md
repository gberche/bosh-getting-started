# Getting a public stemcell and uploading it

A Stemcell is a VM template with an embedded bosh agent. They are the base image for new VMs. That is, on AWS they are the AMI.

Stemcells are large. 400Mb or more. So, run all the following commands from within your BOSH VM where it will be much faster to download and upload the stemcell to your BOSH.

```
$ ssh ubuntu@ec2-10-2-3-4.compute-1.amazonaws.com
sudo su -
gem install bosh_cli
bosh target localhost:25555
bosh public stemcells
+-------------------------------+-----------------------------------------------------+
| Name                          | Url                                                 |                                                                                                                                       +-------------------------------+-----------------------------------------------------+
| bosh-stemcell-0.4.7.tgz       | https://blob.cfblob.com/rest/objects/4e4e78bc...... |
| bosh-stemcell-aws-0.5.1.tgz   | https://blob.cfblob.com/rest/objects/4e4e78bca..... |
+-------------------------------+-----------------------------------------------------+
```

You want the latest public AWS stemcell. Download it from the public server and then upload it to your BOSH:

```
$ cd /tmp
$ bosh download public stemcell bosh-stemcell-aws-0.5.1.tgz
bosh-stemcell:  98% |ooooooooooooooooooooooooooooooo  | 384.0MB   1.7MB/s ETA:  00:00:03
```

If you're not running on the default us-east-1a zone, you may need to update the kernel id to one that is available in your region (preferably matching pv-grub-hd0_1.02-amd64.gz.manifest.xml which is the old 1.02 amd 64 bit kernel). Untar bosh-stemcell-aws-0.5.1.tgz and edit the stemcell.MF (see tips from Richard Lennox https://groups.google.com/a/cloudfoundry.org/forum/?fromgroups#!topic/bosh-users/5Tqw9nK9LtU)

```
mkdir bosh-stemcell-aws-0.5.1
cd bosh-stemcell-aws-0.5.1
tar xvfz ../bosh-stemcell-aws-0.5.1.tgz
vim stemcell.MF
#edit kernel_id
```

add the kernel_id: "aki-fe1354ac" (with the kernel id for your region matching amd64, see http://docs.amazonwebservices.com/AWSEC2/latest/UserGuide/UserProvidedkernels.html and http://cloud-images.ubuntu.com/releases/oneiric/release/published-ec2-release.txt.orig )

```
tar cvfz ../bosh-stemcell-aws-0.5.1.tgz.patched image stemcell*
cd ..
mv bosh-stemcell-aws-0.5.1.tgz bosh-stemcell-aws-0.5.1.tgz.orig
mv bosh-stemcell-aws-0.5.1.tgz.patched bosh-stemcell-aws-0.5.1.tgz
```




```
$ bosh upload stemcell bosh-stemcell-aws-0.5.1.tgz
Verifying stemcell...
File exists and readable                                     OK
Manifest not found in cache, verifying tarball...
Extract tarball                                              OK
Manifest exists                                              OK
Stemcell image file                                          OK
Writing manifest to cache...
Stemcell properties                                          OK

Stemcell info
-------------
Name:    bosh-stemcell
Version: 0.5.1

Checking if stemcell already exists...
No

Uploading stemcell...
bosh-stemcell: 100% |ooooooooooooooooooooooooooooooooo | 389.4MB  37.7MB/s Time: 00:00:10
Tracking task output for task#3...

Update stemcell
  extracting stemcell archive (00:00:06)                                                            
  verifying stemcell manifest (00:00:00)                                                            
  checking if this stemcell already exists (00:00:00)                                               
  uploading stemcell bosh-stemcell/0.5.1 to the cloud (00:06:24)                                    
  save stemcell: bosh-stemcell/0.5.1 (ami-a213cbcb) (00:00:00)                                      
Done                    5/5 00:06:30                                                                

Task 3: state is 'done', took 00:06:30 to complete
Stemcell uploaded and created
```

If you look in AWS console, you'll see an AMI created!

![ami](https://img.skitch.com/20120414-gm2jm4g777mjb6xua68aj1kj43.png)

BOSH knows about your uploaded stemcell (an AMI on AWS):

```
$ bosh stemcells

+---------------+---------+--------------+
| Name          | Version | CID          |
+---------------+---------+--------------+
| bosh-stemcell | 0.5.1   | ami-a213cbcb |
+---------------+---------+--------------+
```
