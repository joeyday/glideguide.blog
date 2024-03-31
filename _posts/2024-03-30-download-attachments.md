---
layout: post
title: Python Script for Downloading Attachments
author: Joey
date: 2024-03-30
categories:
  - python
  - attachments
---

I’ve been exporting a lot of data from my ServiceNow instance lately for archival purposes (<abbr>IYKYK</abbr>) and one of the challenges I’ve encountered is properly preserving attachments.

## Converting from Base64?

At first I looked for ways to “re-hydrate” the attachments from records exported in <abbr>XML</abbr> format. Attachment files are embedded in <abbr>XML</abbr> exports in a kind of [Base64](https://en.wikipedia.org/wiki/Base64) encoding that’s probably not proprietary to ServiceNow, but I tried several different ways to convert that encoded data back into files with no luck. If you know a way to do this, please reach out and share, as I’m still curious to know if it can be done.

## Server-side script and/or Processor?

The next approach I took was writing a server-side script (like a Fix Script or a Background Script) to initiate bulk downloading of attachments in my browser. I stumbled across a few promising Community posts going in a similar direction, including one that suggested creating a <abbr>UI</abbr> Action that utilized a custom [Processor](https://docs.servicenow.com/bundle/washingtondc-application-development/page/script/processors/concept/c_Processors.html). That and similar answers all seemed to be linking to a now-missing post on the [ServiceNow Guru](https://servicenowguru.com) blog.

There’s apparently an out-of-box Processor called DownloadAllAttachmentsProcessor that will (what it says on the tin) download all attachments for a single record as a zip file. Handy! But I needed something that would download attachments in bulk, and custom Processors are not only deprecated but completely disabled (you need the _maint_ role to create them and you can’t edit the <abbr>ACL</abbr>s) so all this was a dead end.

## Attachment <abbr>API</abbr> and Python to the rescue

Fortunately, all that thinking about Processors got me on the related subject of <abbr>REST</abbr> <abbr>API</abbr>s. At first I thought I might have to create a Scripted <abbr>REST</abbr> <abbr>API</abbr>, but then I discovered there is an out-of-box [Attachment <abbr>API</abbr>](https://docs.servicenow.com/bundle/washingtondc-api-reference/page/integrate/inbound-rest/concept/c_AttachmentAPI.html). The last missing piece fell into place when I found [Hitoshi Ozawa using a Python script](https://www.servicenow.com/community/developer-forum/download-file-using-rest-api/m-p/1824873#M481799) to hit the Attachment <abbr>API</abbr> from the command line on his local machine.

I liked Ozawa’s approach, but he was still only downloading the attachments on a single record. So here’s my version of the script that downloads all attachments for all records in a specified table matching a specified encoded query, organizing them in a nice folder hierarchy (don’t miss some notes on usage after the script, including a very important one about security):

~~~ python
import requests
import os
import re

### set options here ###

instance = 'https://<your-instance>.service-now.com'
username = ''
password = ''

table = 'task'
display_field = 'number'
query = 'active=true'
limit = '5' # set to 'None' (without quotes) to remove limit

directory = './Attachments'

### shouldn't have to edit anything below ###

mime_type_to_ext = {
  'application/json': ['.json'],
  'application/xml': ['.xml'],
  'image/png': ['.png'],
  'image/jpeg': ['.jpg', '.jpeg'],
  'text/csv': ['.csv'],
  'text/plain': ['.txt'],
  'application/pdf': ['.pdf'],
  'application/msword': ['.doc'],
  'application/vnd.openxmlformats-officedocument.wordprocessingml.document': ['.docx'],
  'application/vnd.ms-excel': ['.xls'],
  'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet': ['.xlsx'],
}

# helper function for hitting any endpoint, returns response object
def get_response(endpoint, mime_type, params={}):
  headers = { "Accept": mime_type }
  response = requests.get(endpoint, auth=(username, password), headers=headers, params=params)
  if response.status_code != 200:
    print('Status:', response.status_code, 'headers:', response.headers, 'Error Response:', response.json())
    exit()
  return response

# helper function for table api, returns list of dicts
def get_records(table, query, limit=None):
  endpoint = f'{instance}/api/now/table/{table}'
  params = {
    'sysparm_query': query,
    'sysparm_display_value': 'true',
  }
  if limit:
    params['sysparm_limit'] = limit
  
  return get_response(endpoint, 'application/json', params).json().get('result')

# helper function for attachment api, writes files to disk
def get_attachment(sys_id, mime_type, path):
  endpoint = f'{instance}/api/now/attachment/{sys_id}/file'
  response = get_response(endpoint, mime_type)
  with open(path, 'wb') as f:
    for chunk in response.iter_content(chunk_size=1024):
      if chunk:
        f.write(chunk)
        
# helper function to make directory and file names safer
def sanitize_filename(filename):
  filename = re.sub(r'[<>:"/\\|?*]', '', filename) # remove invalid characters
  filename = re.sub(r'^\.+', '', filename) # remove leading period
  return filename[:255] # truncate to 255 characters


# call table api to get specified table records and iterate over results
for record in get_records(table, query, limit):
  class_name = sanitize_filename(record.get('sys_class_name'))
  display_value = sanitize_filename(record.get(display_field))
  print(f'{class_name}: {display_value}')
    
  # call table api again to get attachment metadata and iterate over results
  attachment_query = f"table_sys_id={record.get('sys_id')}"
  attachments = get_records('sys_attachment', attachment_query)
  
  if attachments:
    # make folders if they don't exist
    path = os.path.join(directory, class_name, display_value)
    os.makedirs(path, exist_ok=True)
  
    for attachment in attachments:
      # get mime type and file name
      mime_type = attachment.get('content_type')
      file_path = os.path.join(path, sanitize_filename(attachment.get('file_name')))

      # Append file extension to file name if not already
      file_ext_list = mime_type_to_ext.get(mime_type, [''])
      file_ext = file_ext_list[0]  # Choose first extension as default
      if not any(file_path.endswith(ext) for ext in file_ext_list):
        file_path += file_ext

      # call attachment api to download file and write to disk
      get_attachment(attachment.get('sys_id'), mime_type, file_path)
~~~

## Usage notes

Here’s how to use the script:

1. Copy and paste it into your favorite text editor
2. Update the settings at the top of the script to your liking
3. Save it as _attachments.py_ (or whatever you want)
4. On the command line, navigate to the same folder where you saved the script
5. Run it with the command `python attachments.py` (you might have to start the command with _python3_ depending on your environment)

When you run the script, it creates an “Attachments” folder in the same location. It then begins downloading the attachments into that folder, neatly organizing them in a folder hierarchy by table name and record display value. It also helpfully adds appropriate file extensions to any files that don’t have them.

_I can’t put a fine enough point on this_: It’s super dangerous to put your ServiceNow username and password directly in a plain text file like this. If you’re using it as a one-off like I am, it may be fine, though I’d be sure to delete the password from the script (or just delete the whole script) when you’re done. If this script is going to be used on a regular basis you should investigate dynamically populating the username and password from environment variables or an encrypted password vault. Also, if you can get what you need from a sub-production instance, it’s always safer to go that route. If my script gets you reprimanded or fired over some kind of security breach, don’t @ me.

Lastly, you’ll (obviously) need [Python](https://www.python.org) installed on your local machine to run this. Instructions for doing that are outside the scope of this article, partly since they’ll be different depending on your operating system. In addition to Python, I found I had to separately install the _requests_ package using the [package installer for Python (pip)](https://packaging.python.org/en/latest/tutorials/installing-packages/). Google is your friend for figuring out how to get everything installed and running on your computer.

## Questions or feedback?

If you find a use case for downloading attachments in bulk, I hope the script helps. I’d love to hear about your experience and answer any questions you might have. Feel free to reach out on [SNDevs Slack](https://invite.sndevs.com/) or [Mastodon](https://social.sndevs.com/). Cheers!{% include endmark.html %}

