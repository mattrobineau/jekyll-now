---
layout: post
title: Installing .Net Core version 1.0.5/1.1.2 from tarball
---

Scott Hanselman recently blogged [Trying .NET Core on Linux with just a tarball (without apt-get)](https://www.hanselman.com/blog/TryingNETCoreOnLinuxWithJustATarballWithoutAptget.aspx).
One of the comments posted on the article was by `Matt` (not me):

![FYI, all the tarball links point to v1.0.4 instead of v1.0.5 or v1.1.2]({{ site.url }}/images/posts/2017-6-9/Comment_On_Post.PNG)

Matt's comment is not entirely true. All the SDK links are v1.0.4. However, all the runtime tarballs are v1.0.5.

![Highlighted Runtime download list]({{ site.url }}/images/posts/2017-6-9/net_core_tarball_list.PNG)

Scott does a very good job explaining how to install .Net Core from a tarball. The post was written about a concern about wanting to have
.Net Core installed in one location rather than it going everywhere. Scott's instructions on dealing with this are good but I would like
to point out that Microsoft's own instructions are also valid with one exception. 

The exception being that Microsoft's documentation still points to a v1.0.4.

![Microsoft download instructions -- debian]({{ site.url }}/images/posts/2017-6-9/microsoft_install_instructions.PNG)

The Microsoft documentation should be updated to:

```bash
sudo apt-get install curl libunwind8 gettext
curl -sSL -o dotnet.tar.gz https://download.microsoft.com/download/2/4/A/24A06858-E8AC-469B-8AE6-D0CEC9BA982A/dotnet-debian-x64.1.0.5.tar.gz
sudo mkdir -p /opt/dotnet && sudo tar zxf dotnet.tar.gz -C /opt/dotnet
sudo ln -s /opt/dotnet/dotnet /usr/local/bin
```

You can change the curl string to get v1.1.2 if that is what you prefer:

```bash
curl -sSL -o dotnet.tar.gz https://download.microsoft.com/download/D/7/A/D7A9E4E9-5D25-4F0C-B071-210CB8267943/dotnet-debian-x64.1.1.2.tar.gz
```

This bash script is just like what Scott is doing. It will install .Net Core runtime in `/opt/dotnet`, which you then link to `/usr/local/bin`
so your user can run the `dotnet` command.

Hopefully this clears up how to get the latest (LTS/Current) .Net Core version installed. (As of today)
