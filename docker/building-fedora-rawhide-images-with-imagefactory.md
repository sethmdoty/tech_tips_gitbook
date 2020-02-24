# Building Fedora Rawhide images with Imagefactory

Introduction ImageFactory is a tool, built on Oz, that is suitable for generating various types of operating system and cloud images. These images can be generated in a variety of different formats. Those include the Docker image format and the qcow2 image format. This article shows you how to build a Fedora Rawhide image in Imagefactory, then run it in a container via Docker. In the next article, you’ll see how to do the same thing, except with building qcow2 images for VMs. Why Build a Fedora Rawhide Image? Fedora Rawhide is, as you know, a constantly evolving version of the Fedora operating system that has nightly updates. While you can technically upgrade your OS to Rawhide, most users prefer to stick to the latest Fedora release because Rawhide is not 100% guaranteed to be stable. Still, you may want to test various tools and software on Rawhide, or even play around with Rawhide to see what it has to offer. In these cases, you will likely find container images and VMs useful and appealing. About ImageFactory This section gives a brief overview of ImageFactory and how it works. This is not a comprehensive overview of the tool. Instead, the main purpose of this section is to get you comfortable with how this tool works on a high level. Prerequisites Before we begin, make sure that you have installed a working version of ImageFactory. If you are unsure how to install and setup ImageFactory, $ sudo dnf install imagefactory imagefactory-plugins-TinMan imagefactory-plugins-Docker What do I need to build an image? You will need two files: a kickstartfile and atemplate. The Kickstart fileis used to tell ImageFactory what to install and how to install it. Below is the official Kickstart file for creating a \(minimal\) Fedora docker base image: [https://git.fedorahosted.org/cgit/spin-kickstarts.git/tree/fedora-docker-base.ks](https://git.fedorahosted.org/cgit/spin-kickstarts.git/tree/fedora-docker-base.ks) To download the file to your current directory, $ wget https:git.fedorahosted.orgcgitspin-kickstarts.gittreefedora-docker-base.ks The template is used to tell ImageFactory which OS you want to install \(version, architecture, etc.\) and where to install it from. It essentially describes what to build. An example template can be seen below:

fedora-rawhideFedora26x86\_64https://dl.fedoraproject.org/pub/fedora/linux/development/rawhide/Everything/x86\_64/os/password

Templates can have either a .tdl or a .xml extension. \(Note: You may want to change the root password to something more secure.\) Modifying Oz to work with ImageFactory Since ImageFactory is built on Oz and you may not have enough RAM available in Oz to generate an image, you will have to modify your Oz configuration, which is located at etcozoz.cfg/. Here is the default oz.cfg file:

\[paths\] output\_dir = /var/lib/libvirt/images data\_dir = /var/lib/oz screenshot\_dir = /var/lib/oz/screenshots sshprivkey = /etc/oz/id\_rsa-icicle-gen

\[libvirt\] uri = qemu:///system image\_type = raw type = kvm bridge\_name = virbr0 cpus = 1 memory = 1024

\[cache\] original\_media = yes modified\_media = no jeos = no

\[icicle\] safe\_generation = no

Edit the line which says \#memory = 1024 and change the value to 2048. That should be sufficient to build an image. You can run this command to replace the line: $ sudo sed -i -e ’s/\# memory = 1024/memory = 2048/‘ /etc/oz/oz.cfg Building a Docker container image The general command for building a base image is:

## imagefactory —debug base\_image —file-parameter install\_script   —parameter offline\_icicle true

* The_install\_script_ parameter is self explanatory — it tells Imagefactory which files it should use to build your image with.
* The _offline\_icicle_ parameter tells Oz \(via Imagefactory\) to use features of libguestfs to mount the image offline, chroot into it, then execute an RPM command. \(Normally, Oz derives this information by launching a throwaway version of your image, ssh-ing into it, then running an RPM command. However, because we are building a container image, the actual output is not something that can be booted as a VM, which is why we must derive the ICICLE “offline”.\)

  This process will take more than 10 minutes to run if you do not have a cached copy of the rpms in ImageFactor. Otherwise, it should take anywhere from 5-10 minutes to complete.

  Once the process is finished, it prints the ID of the image you just created. The final output will look something like this:

============ Final Image Details ============ UUID: 17206f41-5bd8-4578-84b9-a3fffc1cd168 Type: base\_image Image filename: /var/lib/imagefactory/storage/17206f41-5bd8-4578-84b9-a3fffc1cd168.body Image build completed SUCCESSFULLY!

Note:If the image build fails, then most likely Rawhide is broken — in which case, you may want to run the tutorial using the latest Fedora release. At the time of this article, that’s Fedora 24. The URL for the F24 repository is [https://dl.fedoraproject.org/pub/fedora/linux/releases/24/Everything/x86\_64/os/](https://dl.fedoraproject.org/pub/fedora/linux/releases/24/Everything/x86_64/os/). You can replace this URL with the URL in the template above. \(Be sure to change the Fedora release number, etc. so your image information is accurate!\) On the other hand, if you receive an SELinux error, then try running setenforce 0 in your terminal. Preparing the image for Docker Next, we need to “docker compress” our image in order to prepare it for loading into Docker:

## imagefactory —debug target\_image —id  docker —parameter compress xz

This time, replace _UUID_ with the UUID printed out, not the Image filename. Once this process completes, you now have your Docker base image! To load your new image into Docker: docker load -i full/path/to/compressed/image/filename

