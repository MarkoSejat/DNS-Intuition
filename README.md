<p align="center">
<img src="https://cdn-icons-png.flaticon.com/512/1183/1183595.png" alt="DNS"/>
</p>

<h1>On-Building Intuition for DNS</h1>
This tutorial is a continuation of the last one where we set up and configured on-premises Active Directory within Azure VMs. A DNS was automatically created when we set up our Active Directory. The purpose of this lab is to familiarize us with DNS, A-Records, CNAME records, and Cache.<br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- DNS

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Inspecting A-Records on server (hostname to IP Address mapping)
- Creating our own A-Records and observing from client
- Deleting records from server and inspect/clear cache to gain an understanding
- Touch on CNAME records (mapping one name to another name)
- Discuss Root Hints. 

<h2>A-Records Exercise</h2>

<p>
<img src="https://i.imgur.com/ElEJnSW.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/ZyHETxh.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
Before we do anything, we want to make sure that we are logged into Client-1 and DC-1 from the MyDomain.com\Jane_Admin account that we created in the previous tutorial. We can then go into Client-1, go to command, and try to ping the computer named "mainframe" (this is a random name I chose). "Mainframe" does not exist anywhere but let us try to ping it anyway and watch it fail. So to understand what happened, we need to understand that our Client-1 computer first tried to find "mainframe" within its local DNS Cache. When it doesn't find anything in there, it wil then try to check its local host file which also showed no results. The last resort is to ask the local DNS server. Because the DNS server also has no idea as to what "mainframe" is, we get the fail message: "Ping request could not find host mainframe. Please check the name and try again". 
</p>
<br />

<p>
<img src="https://i.imgur.com/vXBHBMQ.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/frx2VY8.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/G15Cmaj.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
We are now going to create an A-Record for Mainframe.Mydomain.com and we are going to assign it DC-1's IP Address. We will first go back into our DC-1 VM and go into our Server Manager Dashboard and click on Tools from the top right. Then we will select DNS. A window for DNS Manager will pop up and there we can click on DC-1 from the lefthand side. We should see two files called "Forward Lookup Zone" and "Reverse Lookup Zone". "Forward Lookup Zone" refers to hostname to IP address mapping while "Reverse Lookup Zone" refers to IP address to hostname mapping. We will look at our Forward Lookup Zone and see a list of all of our A-Records. So what are A-Records? An A record maps a domain name to the IP address (Version 4) of the computer hosting the domain. An A record uses a domain name to find the IP address of a computer connected to the internet. The A in A record stands for Address. Whenever you visit a web site, send an email, connect to Twitter or Facebook, or do almost anything on the Internet, the address you enter is a series of words connected with dots. For example, to access the DNSimple website you enter www.google.com. At Google's name server, there’s an A record that points to an IP address. This means that a request from your browser to www.google.com is directed to the server with that specific IP address. A Records are the simplest type of DNS records, and one of the primary records used in DNS servers. You can do a lot with A records, including using multiple A records for the same domain in order to provide redundancy and fallbacks. Additionally, multiple names could point to the same address, in which case each would have its own A record pointing to that same IP address. To put it simply, all A-records really are is just hostname to IP address mapping. Now that we know what A-Records are, lets move on. If we look at the image above, we can see the A-Record for Client-1 and DC-1. We can now go ahead and manually add an A-Record for "Mainframe" by right-clicking and selecting "New Host (A to AAAA..)" and typing in "mainframe" in the next window. We can give it any IP address we want but for this tutorial I am just going to give it  DC-1's IP address thats 10.0.0.4 just so that I can ping it easier. What we are doing here is arbitratry and the purpose of this is just to show what will happen on Client-1 when you attempt to ping Mainframe next time around. This will help us better understand how DNS works. We can go ahead and add it and we should get a pop-up window telling us that we have successfully created the A-Record for mainframe. 
</p>
<br />

<p>
<img src="https://i.imgur.com/7oxWevI.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/XVf6gf4.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
Now if we go back to Client-1 and try to ping "mainframe" we can see that we are succesfully able to do so. We are able to ping 10.0.0.4 which is actually DC-1's IP address that Mainframe is connected to. We can even type in "nslookup mainframe" and we can see that it successfully pings mainframe.mydomain.com and displays the 10.0.0.4 IP.
</p>
<br />

<h2>Local DNS Cache Exercise</h2>

<p>
<img src="https://i.imgur.com/kAi0AEt.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/Aog5gi1.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/xXLhcbG.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
Let us now get into talking about DNS cache. DNS cache refers to the temporary storage of information about previous DNS lookups on a machine's OS or web browser. Keeping a local copy of a DNS lookup allows your OS or browser to quickly retrieve it and thus a website's URL can be resolved to its corresponding IP much more efficiently. If we go to command in Client-1 and type in "ipconfig /displaydns" we will be able to see our entire DNS cache on Client-1. Moving onto the next step, we want to go back into DC-1 and change the mainframe A-Record once more just for the sake of getting a better understanding of DNS. We want to go back into our A-Records and change the IP address of Mainframe to 8.8.8.8, ping it, and see what happens.  
</p>
<br />


<p>
<img src="https://i.imgur.com/M452Hwk.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
We see that if we type in "ipconfig /displaydns" that the old IP address still shows. This is because it is still stored in the cache. This can sometimes cause connectivity issues and we will need to flush all the information out of the cache. We will need to go back into our command prompt but this time we need to run it as administrator since the next step will require admin elevation. If we go ahead and type in "ipconfig /flushdns" and press enter, we can see that all the information is erased.
</p>
<br />


<p>
<img src="https://i.imgur.com/ca4JDxc.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
If we go ahead and try to ping mainframe once more, we can see that we are now getting the updated 8.8.8.8 IP. This is because the cache was wiped clean and Client-1 then checked the Local Host File where it found nothing. Finally Client-1 then asked the DNS for the IP address and we were able to get a  successfull ping.  
</p>
<br />

<h2>CNAME Record Exercise</h2>

<p>
<img src="http://www.marinamele.com/wp-content/uploads/2014/06/dns_table.png" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
At this point we can get into CNAME records. The ‘canonical name’ (CNAME) record is used in lieu of an A record, when a domain or subdomain is an alias of another domain. All CNAME records must point to a domain, never to an IP address. Imagine a scavenger hunt where each clue points to another clue, and the final clue points to the treasure. A domain with a CNAME record is like a clue that can point you to another clue (another domain with a CNAME record) or to the treasure (a domain with an A record). For example, suppose blog.example.com has a CNAME record with a value of ‘example.com’ (without the ‘blog’). This means when a DNS server hits the DNS records for blog.example.com, it actually triggers another DNS lookup to example.com, returning example.com’s IP address 
via its A record. In this case we would say that example.com is the canonical name (or true name) of blog.example.com. Oftentimes, when sites have subdomains such as blog.example.com or shop.example.com, those subdomains will have CNAME records that point to a root domain (example.com). This way if the IP address of the host changes, only the DNS A record for the root domain needs to be updated and all the CNAME records will follow along with whatever changes are made to the root. A frequent misconception is that a CNAME record must always resolve to the same website as the domain it points to, but this is not the case. The CNAME record only points the client to the same IP address as the root domain. Once the client hits that IP address, the web server will still handle 
the URL accordingly. So for instance, blog.example.com might have a CNAME that points to example.com, directing the client to example.com’s IP address. But when the client actually connects to that IP address, the web server will look at the URL, see that it is blog.example.com, and deliver the blog page rather than the home page. 
</p>
<br />


<p>
<img src="https://i.imgur.com/NnyhVEh.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
For this tutorial, I will be making "search" my CNAME. So let us go back to Client-1 and try to ping search from our command prompt just to see what happens. We see that it tells us that it could not find the host search.  
</p>
<br />


<p>
<img src="https://i.imgur.com/xYDBCCN.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/zfPOuey.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
We can now go back into DC-1's VM and go back into the DNS Manager inside of our Server Manager. From there we can right-click mydomain.com and go to New CNAME. From there we can give it the Alias name "search" and put www.google.com as the FQDN (Fully Qualified Domaine Name) for target host. This will have no real utility. The purpose of doing this is just for demonstrating that you can map names to other names. We can now go back to Client-1 and try to ping search again.   
</p>
<br />


<p>
<img src="https://i.imgur.com/i390TIj.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
If we flush the cahce, ping search again and displaydns in command, we can see that the record name of search.mydomain.com has a CNAME record of google.com. We can also see that google gives us its A Record IP. Search.mydomain.com resolves to google and then google resolves to A (host) Record of 142.250.189.4 in the example above. We can now see how A-Records and CNAME works. We also got to see how the DNS Cache works and why sometimes we might need to flush it. 

</p>
<br />

<h2>Root Hints</h2>

<p>
<img src="https://i.imgur.com/7wYY06Y.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
We know that Client-1 is relying on DC-1 to resolve all domain queries. We know that Client-1 can only go to the domain names that are either stored in its cache, local host, or the DNS so how is it that we are able to go to sites like google.com or microsoft.com without having them manually entered into our DNS? This is done through Root hints. Root hints are a list of the DNS servers on the Internet that your DNS servers can use to resolve queries for names that it does not know. When a DNS server cannot resolve a name query by using its local data, it uses its root hints to send the query to a DNS server. 
</p>
<br />


<p>
<img src="https://i.imgur.com/rCExmvh.jpg" height="100%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
So for example, if we were to go to a site like www.azure.microsoft.com and then go to command and type in "ipcong /displaydns", I will get a bunch of stuff coming up on the back end. Furthermore, If I type in "ipconfig /all", I can see that our Client-1 is relying solely on DC-1 to resolve all these names. This is all because of root hints. We can open up Root Hints by going to: DNS Manager > DC-1 > Right-click DC-1 > properties > Root Hints. From there we can see a list of all root hints servers that it uses to resolve all these different domains. I hope that this gives you a better understanding of DNS and root hints. Thank you!
</p>
<br />
