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
3. We executed the exploit with the target's URL and ID 1 (likely the ADMIN ID).
4. We obtained 3 possible URLs, and one of them granted us administrator privileges.
5. We accessed the provided posts section at http://ctf-fsi.fe.up.pt:5001/wp-admin/edit.php.
6. In an admin post, we found the flag.

## Identification

- CVE-2021-34646 - WordPress Plugin WooCommerce Booster Plugin 5.4.3 - Authentication Bypass (https://www.exploit-db.com/exploits/50299)

- This vulnerability allows remote attackers to bypass authentication and act as other users. The WooCommerce Booster Plugin has a weak random token generation in the reset_and_mail_activation_link function found in the ~/includes/class-wcj-emails-verification.php file. Attackers can easily generate a token that, when used with the process_email_verification function, allows the impersonation of users, including administrators. This vulnerability requires the "Email Verification" module and the "Login User After Successful Verification" setting to be enabled, which it is by default.

- Affected systems:
    - WordPress Plugin WooCommerce Booster Plugin 5.4.3 (inclusive)

## Catalogue

- This exploit garnered an exceptionally high CVSS score of 9.8 out of 10, signifying its extreme criticality.

## Exploit

The exploit used by us was found in the following url: https://www.exploit-db.com/exploits/50299

It's a python script that takes a URL and a user ID as parameters. It, then, generates possible email verification tokens for that user, through the known weak token generation method. Finaly, it creates various URLs with those tokens, of which, one of them can authenticate the attacker as the user for the given URL.

## Atacks


