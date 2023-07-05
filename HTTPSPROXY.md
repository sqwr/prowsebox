We describe here how to configure a browser HTTPS Proxy settings. In most cases, this boils down to adding the custom Certificate Authority (CA) of the Proxy to the browser or device list of trusted CAs.  

We experimented with the following HTTPS proxies. 
1. [Mitmproxy](https://mitmproxy.org/): a Python scriptable HTTPS proxy
2. [Mockttp](https://github.com/httptoolkit/mockttp): a Node.js scriptable HTTPS proxy 

We have performed most of the experiments and analyses using Mitmproxy. At this point, we are only publishing this option. We will release the other option when it is more extensively tested. 

# Mitmproxy
We recommend reading the [docs](https://docs.mitmproxy.org/stable/overview-installation/) to install Mitmproxy. We provide here examples instructions for mostly GNU Linux and Mac OSX platforms where we have run most of these experiments. These instructions can also be found in the [Installation Guide](INSTALL.md). 

We recommend version 9.0.1, the last version successfully tested to work with the framework.



## GNU/Linux systems
```bash
# Download 
curl -o mitmproxy.tar.gz https://snapshots.mitmproxy.org/9.0.1/mitmproxy-9.0.1-linux.tar.gz
tar xvf mitmproxy.tar.gz 
mkdir -p ~/.local/bin
mv mitmdump mitmproxy mitmweb ~/.local/bin/
echo "export PATH=:$HOME/.local/bin:$PATH" >> ~/.bashrc
source ~/.bashrc
rm -rf ~/.mitmproxy
mitmdump --version
timeout 5 mitmproxy
```



# Mockttp
[Mockttp](https://github.com/httptoolkit/mockttp/) is an alternative to Mitmproxy. If you are interested in porting the the [src/mitmproxies/http]


