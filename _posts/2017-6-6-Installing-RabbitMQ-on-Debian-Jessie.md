---
layout: post
title: Installing RabbitMQ on Debian Jessie
---

I have recently found myself in need of a message broker for my current project. After a bit of research of the existing implementations
of message brokers and messaging queues in general, I have settled on using RabbitMQ.

<h2>Simplest way to install RabbitMQ</h2>
Direct from the (RabbitMQ Documentation)[https://www.rabbitmq.com/install-debian.html].
We'll be using `sudo` here. If you do not have it installed, you can either install it and give your
user rights, or run the commands (after `sudo`) as root. It is recommended to use sudo, but I wont be angry if you don't.

{% highlight bash %}
echo "deb http://www.rabbitmq.com/debian/ testing main" |
     sudo tee /etc/apt/sources.list.d/rabbitmq.list
     
wget -O- https://www.rabbitmq.com/rabbitmq-release-signing-key.asc |
    sudo apt-key add -
    
sudo apt-get update
sudo apt-get install rabbitmq-server
{% endhighlight %}

And that's pretty much it. 

Make sure to (configure)[https://www.rabbitmq.com/configure.html] your new `RabbitMQ` install.
