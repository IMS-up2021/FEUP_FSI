<<<<<<< HEAD
# CTF - Wordpress CVE
=======
# Trabalho Realizado na Semana #3

## CTF - Wordpress CVE

### Desafio 1
>>>>>>> 06b488922b8edc81efea43e4a2ab79052f8baef5

While searching the website, we found the versions of the plugins used in the website in the following page:

`http://ctf-fsi.fe.up.pt:5001/product/wordpress-hosting/`

<<<<<<< HEAD
When selecting the 'Additional information' tab.
=======
When selecting the 'Additional information' tab, which were WooCommerce and Booster for WooCommerce.

We also found the active users in the same page, when selecting the 'Reviews' tab, which were Orval Sanford and admin.
>>>>>>> 06b488922b8edc81efea43e4a2ab79052f8baef5

Upon getting information on the plugins and their versions, we started to search for potential vulnerabilities that could allow us to enter any account registered in the website.
Doing our research, we came across CVE-2021-34646, with a CVSS score of 9.8, whose description say "Versions up to, and including, 5.4.3, of the Booster for WooCommerce WordPress plugin are vulnerable to authentication bypass via the process_email_verification (...) This allows attackers to impersonate users and trigger an email address verification for arbitrary accounts (...)".
Given that the challenge's "Booster for WooCommerce plugin"'s version was 5.4.3, we submitted the CVE code to Challenge 1.

<<<<<<< HEAD
=======
### Desafio 2

>>>>>>> 06b488922b8edc81efea43e4a2ab79052f8baef5
After discovering the CVE, we started looking for exploits that took advantage of it to get access to a privileged account in the website, and found it in the following page:

`https://www.exploit-db.com/exploits/50299`

<<<<<<< HEAD
After running the script, we are provided with 3 links, one of which provides access to the administrator account, from which we can access a link to enter the administrator page and through the first post we find the flag: "please don't bother me"
=======
After running the script by giving the url of the page and the id of the user we want to access the account (in our case it was id 1, since it is the usual id for the admin because most of the time it is the first account to be created), we are provided with 3 links, one of which provides access to the administrator account, from which we can access the link given in moodle (`http://ctf-fsi.fe.up.pt:5001/wp-admin/edit.php`)to enter the administrator page and through the first post we find the flag:{"please don't bother me"}.
>>>>>>> 06b488922b8edc81efea43e4a2ab79052f8baef5
