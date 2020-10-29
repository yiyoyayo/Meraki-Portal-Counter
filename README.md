# Meraki-Portal-Counter

This is a software project designed for a retail customer using Meraki. Project is designed to showcase Meraki API capabilities and is based on assumptions. Please refer to detailed "User Guide.pdf" documentation provided in this repo.


## Use Case Description

The purpose of this project is to provide a solution for the retailer so they can get people traffic information easily. By using the analytical results, retailers have insights of the market trends and customer interests, which contribute in a targeted marketing campaign and successful business strategy.

This project is not a fully working solution. This is more of a proof of concept to illustrates the capabilities of Meraki products and how they help achieve desired outcome of the case study.
This project uses the following:
* Meraki cameras to detect and count people entering and leaving the store
* Meraki access points to detect and count devices in the store
* Meraki Captive Portal to capture the customers’ information, and redirect them to products of interest.
* Kibana to visualize the customer traffic and device information.
* Backend code written in python.

## Software dependencies
* Python 3
* Elasticsearch
* Kibana
* ngrok
* Meraki AP and Meraki Camera

## Installation

* You would need to install all software dependencies listed above
* Setup ngrok
  * [Install ngrok](https://ngrok.com/download)
  * Replace ```$HOME/.ngrok2/ngrok.yml``` with *ngrok.yml* file from this repo.
  * This will allow you to automatically run 4 tunnels you need for the main applications.
  * Note, you will need to register for ngrok if you want to run multiple tunnels. Don't worry, it's free.
  * You should get one ngrok URL for every python script based on the port that script uses. More on this in Description section.
* Install Elasticsearch
  * [On MAC and Linux](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html)
  * [On Windows](https://www.elastic.co/guide/en/elasticsearch/reference/current/zip-windows.html)
* Install Kibana
  * [On Mac and Linux](https://www.elastic.co/guide/en/kibana/current/targz.html)
  * [On Windows](https://www.elastic.co/guide/en/kibana/current/windows.html)
* [Install Python](https://www.python.org/downloads/)
  * 3.x is required
* Install python dependencies
  * Open terminal in root directory of this folder and run following commands.
  * Ensuring you have *requirements.txt* file, run : ```pip3 install -r requirements.txt ```
* Once you have your ngrok URLs you would need to add these to Meraki Dashboard. __For these remaining steps, I would recommend you look at "User Guide.pdf" documentation provided in this repo. It is more comperhensive walkthough with nice pictures__
* You need to get Meraki camera API key and Camera serial number and change the following values in *config.py* file:
  * MERAKI_API_KEY
  * CAMERA_SERIAL
* You need to do similar for location scanning part, for which you need to obtain API Key and "secret" phrase, and update the following values in *config.py* file:
  * VALIDATOR
  * SECRET


## Configuration

Please refer to detailed "User Guide.pdf" documentation provided in this repo.

## Usage

Please refer to detailed "User Guide.pdf" documentation provided in this repo.

## How to test the software

### Start ngrok tunnel
Assuming you have copied *ngrok.yml* in previously mentioned directory, you can just run:
  ```
  ngrok start -all 
  ```
If you have not, or you wish to test each script separetrly you can run single ngrok tunnel:
  ```
  ngrok http <port_number>
  ```
For example, to run tunnel for elasticsearch only, you would do:
  ```
  ngrok http 9200
  ```
Assuming you have not changed default port elastic is running on (i.e. 9200).

#### Why do we need ngrok?

* Elasticsearch, Kibana, *meraki_captive_portal.py* and *location_scanning_v2.py* run as servers. 
* Both meraki_captive_portal.py and location_scanning_v2.py need ngrok tunnel because we need to provide this URL to Meraki Camera and AP.
* Elasticsearch and Kibana can be run locally.
* In fact all these servers are running locally (i.e. localhost). What ngrok does, it provides public URL (i.e. domain) that points to your localhost:<port>, that Meraki Dashboard can access. 
* By default (and you will see this when you look at the code) these are default localhost ports these applications are using:
  * meraki_captive_portal   : __5004__
  * location_scanning_v2.py : __5000__
  * Elasticsearch           : __9200__
  * Kibana                  : __5601__
  
  * NOTE: All python scripts use ngrok URL when writing data to Elasticsearch. This is placed under variable named __ES_HOST_URL__ in *__config.py__* file, so if you re-run ngrok you would need to change URL or change it to localhost:9200 (if you do not change elastic default port of course).
  * Oh yea, since we are using free version of ngrok, everytime you re-run it, it will give you a different URL, so you need to update Meraki Dashboard and *__config.py__* file.

### Initiate Elasticsearch and Kibana

* In Termial, navigate to where you extrcted elasticsearch then type:
```
./bin/elasticsearch
```
* If you want to test it manually, wait for a minute or so, then in your browser go to __localhost:9200__ or __respective ngrok URL__ to see if elastic is up and running. You should get json object displayed. Otherwise if you run your python scripts it will tell you if there is an issue with elastic.

* In a separate terminal, navigate where you extracted kibana then type:
```
./bin/kibana
```
* To test and access your data, in your browser go to __localhost:5601__ or __respective ngrok URL__. Kibana UI should pop up. On how to use Kibana interface refer to our "User Guide.pdf" document included in this repo.

* __NOTE__: If you are running this code for the very first time, once you have elastic up and running, run the following python script:
```
pyhton3 elastic_onetime.py
```
  * You only need to run this once (i.e. when you are setting up elasticsearch for the first time). __This scrip will create all relevant indecies for elasticsearch where you can store your data.__
  
* The following is the mapping of which script uses which elasticsearch index:

Python Script | Elasticsearch Index
------------- | -------------------
camera_count_v2.py | __camera_count__
location_scanning_v2.py | __scanning_count__
meraki_captive_portal.py | __splash_count__

### Run Python Scripts
There are 3 main functions: camera_count_v2, location_scanning_v2.py and meraki_captive_portal.py.
Run each one in a separate terminal.
  ```
  python3 camera_count_v2.py
  ```
  ```
  python3 location_scanning_v2.py
  ```
  ```
  python3 meraki_captive_portal.py
  ```
  # Testing
  * You should be all up and running now.
  * You can test by using camera and AP (i.e. walk into the frame or camera, and/or connect to wifi)
  * Then you can go to ngrok URL that is pointing to Kibana (i.e. localhost:5601) and you will see all of the collected data.
  * Refer to "User Guide.pdf" documentation provided in this repo for more information on how to use Kibana to display your information. Alternatively, you can see demo video.

### Authors Note
* We did not really had much time to create unit test (sorry...) but you can kinda test some scripts without needing Meraki AP.
* Most scripts are dependent on Meraki AP and Camera in order to test their functionality.
* You can test elasticsearch without requirement for any ngrok. Simply run it locally and change ES_HOST_URL to localhost:9200. Similarly you can do this for Kibana too.
* You can test splash page without need for Meraki hardware.
  * Run ``` python3 meraki_captive_portal.py``` and ```ngrok htto 5004``` (ngtok is optional here)
  * Then copy the following string:
  > <your_server>/click?base_url = "base_grant_url=https%3A%2F%2Fn143.network-auth.com%2Fsplash%2Fgrant&user_continue_url=http%3A%2F%2Fmeraki.com%2F&node_id=1301936&node_mac=00:18:0a:13:dd:b0&gateway_id=1301936&client_ip=10.162.50.40&client_mac=44:55:66:77:88:BB
  * Where <your_server> is either localhost:9200 or ngrok URL.
  * You should then be greeted by splash page where you can follow through.
  * Once you submit your response, you will be redirected based on the interests you selected.
  * Either in your terminal or in Kibana (if you are running elastic) you will be able to see information captured from the captive portal.
  
## Known issues
* In the scenario where 1 person has multiple devices, the current version will count it as multiple clients
* The current version cannot solve the random MAC address issue - device will be counted as different device if their MAC address changed

## Getting help

Please refer to detailed "User Guide.pdf" documentation provided in this repo.


----

## Licensing info

                           CISCO SAMPLE CODE LICENSE
                                  Version 1.1
                Copyright (c) 2018 Cisco and/or its affiliates

   These terms govern this Cisco Systems, Inc. ("Cisco"), example or demo
   source code and its associated documentation (together, the "Sample
   Code"). By downloading, copying, modifying, compiling, or redistributing
   the Sample Code, you accept and agree to be bound by the following terms
   and conditions (the "License"). If you are accepting the License on
   behalf of an entity, you represent that you have the authority to do so
   (either you or the entity, "you"). Sample Code is not supported by Cisco
   TAC and is not tested for quality or performance. This is your only
   license to the Sample Code and all rights not expressly granted are
   reserved.

   1. LICENSE GRANT: Subject to the terms and conditions of this License,
      Cisco hereby grants to you a perpetual, worldwide, non-exclusive, non-
      transferable, non-sublicensable, royalty-free license to copy and
      modify the Sample Code in source code form, and compile and
      redistribute the Sample Code in binary/object code or other executable
      forms, in whole or in part, solely for use with Cisco products and
      services. For interpreted languages like Java and Python, the
      executable form of the software may include source code and
      compilation is not required.

   2. CONDITIONS: You shall not use the Sample Code independent of, or to
      replicate or compete with, a Cisco product or service. Cisco products
      and services are licensed under their own separate terms and you shall
      not use the Sample Code in any way that violates or is inconsistent
      with those terms (for more information, please visit:
      www.cisco.com/go/terms).

   3. OWNERSHIP: Cisco retains sole and exclusive ownership of the Sample
      Code, including all intellectual property rights therein, except with
      respect to any third-party material that may be used in or by the
      Sample Code. Any such third-party material is licensed under its own
      separate terms (such as an open source license) and all use must be in
      full accordance with the applicable license. This License does not
      grant you permission to use any trade names, trademarks, service
      marks, or product names of Cisco. If you provide any feedback to Cisco
      regarding the Sample Code, you agree that Cisco, its partners, and its
      customers shall be free to use and incorporate such feedback into the
      Sample Code, and Cisco products and services, for any purpose, and
      without restriction, payment, or additional consideration of any kind.
      If you initiate or participate in any litigation against Cisco, its
      partners, or its customers (including cross-claims and counter-claims)
      alleging that the Sample Code and/or its use infringe any patent,
      copyright, or other intellectual property right, then all rights
      granted to you under this License shall terminate immediately without
      notice.

   4. LIMITATION OF LIABILITY: CISCO SHALL HAVE NO LIABILITY IN CONNECTION
      WITH OR RELATING TO THIS LICENSE OR USE OF THE SAMPLE CODE, FOR
      DAMAGES OF ANY KIND, INCLUDING BUT NOT LIMITED TO DIRECT, INCIDENTAL,
      AND CONSEQUENTIAL DAMAGES, OR FOR ANY LOSS OF USE, DATA, INFORMATION,
      PROFITS, BUSINESS, OR GOODWILL, HOWEVER CAUSED, EVEN IF ADVISED OF THE
      POSSIBILITY OF SUCH DAMAGES.

   5. DISCLAIMER OF WARRANTY: SAMPLE CODE IS INTENDED FOR EXAMPLE PURPOSES
      ONLY AND IS PROVIDED BY CISCO "AS IS" WITH ALL FAULTS AND WITHOUT
      WARRANTY OR SUPPORT OF ANY KIND. TO THE MAXIMUM EXTENT PERMITTED BY
      LAW, ALL EXPRESS AND IMPLIED CONDITIONS, REPRESENTATIONS, AND
      WARRANTIES INCLUDING, WITHOUT LIMITATION, ANY IMPLIED WARRANTY OR
      CONDITION OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, NON-
      INFRINGEMENT, SATISFACTORY QUALITY, NON-INTERFERENCE, AND ACCURACY,
      ARE HEREBY EXCLUDED AND EXPRESSLY DISCLAIMED BY CISCO. CISCO DOES NOT
      WARRANT THAT THE SAMPLE CODE IS SUITABLE FOR PRODUCTION OR COMMERCIAL
      USE, WILL OPERATE PROPERLY, IS ACCURATE OR COMPLETE, OR IS WITHOUT
      ERROR OR DEFECT.

   6. GENERAL: This License shall be governed by and interpreted in
      accordance with the laws of the State of California, excluding its
      conflict of laws provisions. You agree to comply with all applicable
      United States export laws, rules, and regulations. If any provision of
      this License is judged illegal, invalid, or otherwise unenforceable,
      that provision shall be severed and the rest of the License shall
      remain in full force and effect. No failure by Cisco to enforce any of
      its rights related to the Sample Code or to a breach of this License
      in a particular situation will act as a waiver of such rights. In the
      event of any inconsistencies with any other terms, this License shall
      take precedence.

