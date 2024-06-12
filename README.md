# Adoption and Compliance with Standards of the DMARC Reporting System. Artifacts.

This repository contains the data used for generating the table 'Results of the Email Service Provider (ESP) measurements showing their adherence to specifications'. The table is compiled from various sources and services utilized for the measurements.
The LaTeX code of the table can be found in the file 'dmarc_features_table.tex'.

# Table generation

The table is generated from the Jupyter Notebook "GenerateTable.ipynb".

The Jupyter Notebook uses various files in the "generated/" folder. 
Althought they are allready generated, you can re-generate them using the jupyter notebook "GenerateData.ipynb"


# The datasets 

The datasets folders contains multiples files such as follows : 

- datasets/EmailReceived/: Contains all the emails received in the different ESP mailboxes. 
- datasets/ReportReceived/: Contains all the DMARC reports received, according to the RUA/RUF DMARC tags.
- datasets/DNS/: Contains the query.log file with BIND service DNS logs and a directory 
zonefile with zone file of the domain names ['email-sender-1.example',  'email-sender-2.example', 'email-sender-3.example' ,
'email-sender-4.example', 'email-receiver-overwrite.example' ]
- datasets/Rspamd/logs.json contains the Rspamd logs from the mail suite. It allow us to verify the
setup and configuration of the emails sent.



# Domain and IP address anonymization.

Registered domain names that are not part of the ESP ecosystem, as well as IP addresses, have been anonymized.

Anonymized domains follow this pattern:

- TLDs have been replaced by '.example' (following [RFC 2606](https://www.rfc-editor.org/rfc/rfc2606.html#section-2)).
- The first leaf has been hashed and salted.
- Subdomains have been preserved.

For example, the registered domain name 'conext24-june.hotcrp.com' could be replaced by 'conext24-june.ib4h5rqe2gvhcuu4djg.example'.

Both IPv4 and IPv6 addresses have been anonymized and are surrounded by distinguishable tags, such as:

- XX_ANONYMIZED_IP6_scpkb14n0zqsysbt3uk_XX
- XX_ANONYMIZED_IP4_ex2ftnip6aahjmurihk_XX



# The Experiment

Using name server logs and received emails, we collect data on a set of eight features defined as
follows: 
- 𝐹1
(𝐷𝑀𝐴𝑅𝐶𝑟𝑒𝑐𝑜𝑟𝑑𝑡𝑥𝑡 ) verify whether the ESP queries the records
related to the anti-spoofing measures. 
- 𝐹2 (𝐷𝑀𝐴𝑅𝐶ℎ𝑒𝑎𝑑𝑒𝑟)
verify the presence of the authentication information in the
Authentication-Results email headers.
 - 𝐹3 (𝐷𝑀𝐴𝑅𝐶𝑟𝑢𝑎) and 𝐹4 (𝐷𝑀𝐴𝑅𝐶𝑟𝑢𝑓 ) indicate whether the ESPs send the aggregate and failure reports. Features 𝐹5−8 complete the behavior
of ESPs related to EDV and can only be evaluated if the ESP
sends reports. 
- 𝐹5 (𝐸𝐷𝑉𝑞𝑢𝑒𝑟𝑦) indicates if the ESP performed
the EDV
- 𝐹6 (𝐸𝐷𝑉𝑐𝑜𝑛𝑡𝑟𝑜𝑙) is a control feature: it verifies the
delivery of report in external email addresses with a correctly set up infrastructure. Even if ESP does perform EDV,
we should receive the report. 
- 𝐹7 (𝐸𝐷𝑉𝑓𝑎𝑖𝑙𝑢𝑟𝑒) verifies that the
ESP does not send the reports when EDV fails. 
- Finally, 𝐹8 (𝐸𝐷𝑉𝑜𝑣𝑒𝑟𝑤𝑟𝑖𝑡𝑒) verify that reports are sent to the new email
address present in the EDV record.



## Testbench infrastructure 
Measurements were performed on a server running Ubuntu 22.04.3 LTS.
The main services used for the experiment are [Bind9](https://www.isc.org/bind/)
 and the mail suite [mailcow](https://mailcow.email/).

We have sent a total of 16 emails to the email addresses associated to the registered account of 
 each ESPs.
Those 16 emails were sent using 4 different 'From.RFC5322' domain names. For each domain we
have emails with DKIM alignments and SPF alignments following the cartesian products :


{aligned DKIM, not aligned DKIM} x {aligned SPF, not aligned SPF}


### Domain name configuration

4 registered domain names have been used to send emails : 
- email-sender-1.example : DMARC set up with RUA and RUF uri within the same organizational domain
- email-sender-2.example, email-sender-3.example, email-sender-4.example : DMARC setted up with RUA and RUF uri are not within the same organizational domain

According to the RFC 7489, email receiver should proceed the EDV when generating the DMARC reports.
Behavior towards the EDV are 

- email-sender-2.example : Reports are sent to address within the 'control-domain.example' organizational domain name. 
DMARC EDV records are presents. The reports can be sent to those email addresses.
- email-sender-3.example : Reports are sent to email addresses part of 'email-recevier-invalid-edv.example'. The DMARC EDV records are not present. We should not receive any reports.
- email-sender-4.example : Reports are sent to email addresses part of 'email-receiver-overwrite.example'. 
DMARC EDV records are presents with additional RUA tags with a new email address 'redirectionRUA@email-receiver-overwrite.example'.
The reports should be sent on this email address

 To avoid unsolicited DNS requests interfering with our measurements, each ESP had a specific
 subdomain for all the sending emails. As a result, each 'email-sender-x.example' domain names have subdomains set up for each ESP (e.g : fastmailcom.email-sender-1.example, gmailcom.email-sender-1.example, etc.. )



### Networking 
The servers had IPv4 and IPv6 connectivity with a total of 4 ip addresses (anonymized, anonymized, 
anonymized, and  anonymized). All IP addresses contain DNS pointer reversed to the domain name
'mail.mail-server.example'.

# Features evaluation

With the configuration describe above, each DMARC features from Fastmail can be evaluated as follow : 

- F1 : Verify the presence of the DNS query : TXT _dmarc.fastmailcom.email-sender-1.example
- F2 : Verify the presence of the authentication information in the Authentication-Results email headers received in Fastmail's inbox. 
- F3 : Verify the presence of any aggregate reports towards fastmailcom.email-sender-1.example
- F4 : Verify the presence of any failure reports towards fastmailcom.email-sender-1.example
- F5 : Verify the presence of the DNS query : TXT fastmailcom.email-sender-4.example._report._dmarc.email-receiver-overwrite.example
- F6 : Verify the presence of any aggregate reports towards fastmailcom.email-sender-2.example in the control-domain.example inbox
- F7 : Verify the absence of any aggregate reports towards fastmailcom.email-sender-3.example in the email-recevier-invalid-edv.example inbox
- F8 : Verify that aggregate reports towards email-sender-4.example are sent to redirectionRUA@email-receiver-overwrite.example instead of rua@email-receiver-overwrite.example

All those evaluations are made in the Jupyter Notebook GeneratedData.ipynb. 


# Python environement 
The requirements.txt file lists all the necessary libraries to run the Jupyter notebooks. 

Run the following command to install the dependencies

```
pip3 install -r requirements.txt
```

# Remarks 
Even though it does not impact the measurement analysis of the current paper, the content of the email titled "- SPAM orangefr Email Research Testing Dmarc configuration - SPK OK - DKIM NOK- 13 42 16.416362" is lost.


