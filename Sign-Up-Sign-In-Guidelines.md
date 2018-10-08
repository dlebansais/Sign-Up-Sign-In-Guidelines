#Sign Up/Sign In Guidelines

The aim of this document is to describe the complete implementation, from the perspective of security, of a system that allows users to sign up for access, and then sign in/sign out. The term guideline is used because what is given here is pseudo code and examples rather than source code.

The system has been in fact implemented as a proof of concept. Here I will explain all the details relevant to guidelines, but not implementation details, when for instance they depend on the language used or the underlying architecture.

I'm not a security expert, don't rely on that. I've done a bit of research and I hope this system is sound, but if you think there is a flow in the logic, you're most definitely welcome to tell me, and I'll try to fix it. Also, if you can provide a link to other guides of this type I'll gladly add them, or just point people to them if they are better than this one.

##The system
Let's define what I mean by this. The system is a computing facility that performs tasks (unspecified here), and has the concept of users. A new user can introduce themselves by mean of a sign up procedure, described below. Once the user is accepted by the system, they can sign in to obtain access to the computing facility, and sign out to release that access. Finally, they can request to be removed from the system.

This is a fairly common scheme, a typical client/server abstraction.

##Terms
###User
A human (presumably) that wants to use the system.
###Credential
Information associated to the user that allows the system to recognize a valid access request.
###Password
A sentence known to the user only. It is their responsability to keep it secret, and if another human requesting access demonstrates they know the password, the system will not be able to make the difference and will grant the request.
###Database
The storage location for credentials.
###Client
The software used to provide sign up and sign in information to the system.
###Server
The software that checks requests and grant them.
###Username
A name associated to the user, used by the system to communicate wih them.
###Email address
Email is a separate mean of communication with the user. The address is the identifier of the user when sending emails to them.
###Salt
Binary data created during sign up. This data is unique, and public.
###Password settings
All settings related to the encryption of the password.
###Question
An optional free text intended to be a question for which only the user knows the answer.
###Answer
A sentence known to the user only, expected to be deduced from the question years after they forgot their password.
###Answer settings
All settings related to the encryption of the answer.
###Transaction Code
Random binary data that don't need to be cryptographically secure and has a short validity period.
###Transaction end
The date after which a transaction code is no longer valid.
  
##Priorities
Securing a system is a compromise, between usability and protection against attacks. For example, ideally a user would not have to provide a password, they would just show up and have their access granted. On the other hand, from the perspective of the system, a simple password is not very safe, particularly if it doesn't have to meet requirements such as minimum length. For a computer, a page-long password is safer than four digits!

Here I want to state what have been my priorities when designing the system. This is essential because it led me to unusual choices, not recommended by other people commenting on the subject.

1. The secrecy of the password must be protected at all cost.
That is to say, the number one priority is to make sure only the user knows the secret password, and I prefer to grant access to an attacker of the system rather than to risk disclosing the password.
2. The sign in procedure is simple and fast.
In the compromise between usability and security, for signing in, I lean toward the usability side. For example I did not implement two-steps verification. 
3. The sign up procedure, and other operations related to changing credentials, are secure at the expense of the user if needed. Contrary to signing in, signing up is done once. At that time, I think it's acceptable to expect from the user patience and understanding, and to implement a secure scheme.

##Assumptions
I will assume the following:

+ When signing in, the user can provide either their username or their email address. Therefore, I assume uniqueness of both: users can't share either a username or an email address.
+ Username and email address are not kept secret. They are not public in the sense that the system doesn't provide a list of registered usernames or email addresses, but any attacker can check whether a username, or an email address, is taken, by simply trying to sign up with it.
+ The user can change the username, the email address and the password they use after they have signed in. However, the system is allowed to be more secure than fast for this, in the same way than during sign up.
+ To sign up, it is necessary and sufficient that the user controls their email address and has access to the internet (they can click links).
+ To change credentials afterward, it is necessary and sufficient that the user can prove they known the password at the time of the change.

##Optional parts
This guideline include some optional parts, indicated as such. One is the use of a secret question/answer challenge to provide additional security when an email address is compromised, the other is the use of an external site for providing crytographically secure random data.

###Secret question/answer challenge
Despite the many advices against using a secret question/answer challenge, I consider my implementation reasonably safe and useful. Since it's optional you are free to disagree and to just not implement it.

The idea behind this challenge is that, if the user's email address is compromised and an attacker can at least read emails, they can request to recover a lost password and see the recovery email sent by the server. Armed with the link within, they can change the password, the user name and email address, taking total control over the account.

In other words, the credential saved so carefully is no safer than the credential used by the email server. The later might be good enough, or it might be bad. If you don't trust users to choose properly managed email servers, having an additional verification steps might be a good idea.

Implementing it, beside the implementation cost for programmers, could have zero impact on the process from the perspective of the user: they could just leave both the question and answer empty, or fill it with useless values such as foo/foo. It's an additional level of safety left to the user's choice.   

###External source of randomness
You can use an external source to obtain the [salt](#salt). For example, from https://beacon.nist.gov/home.

Pros:

- As cryptographically secure random as you can expect, if you trust this site.
- Any change in the safety and randomness of this value will probably be moved to a different address, so you can expect this source to not change.

Cons:

- Not guaranteed to be available and accessible.
- Not guaranteed to provide a different value for each request.

Whether you implement this or not, you should fall back to a local implementation. I will elaborate on this when describing [sign up](#sign_up).

##Overview
This section gives an overview of the process of signing up/in/out. With it the reader can quickly figure out if it will fit their needs, or if they want something different, in which case there is no point reading the entire guide.
 
1. Sign up
  - The user provides a username, an email address, a password and other informations specific to the site.
  - The server checks if the username or email address is taken.
  - If not, the password is encrypted and an email is sent to the provided address with a link to click.
  - When the user has received the email, clicked the link and confirmed the password, the sign up step is complete.
1. Sign in
  - The user provide a username or email, and a password.
  - The password is encrypted, and if it matches a known credential, by username or email, access is granted.
1. Changing credential
  - The option to change credential is available only to signed in users.
  - To change the username, the email or the pasword, the user is asked to enter their current password and the modified information.
  - The password is encrypted and if it matches the current credential the change is applied
1. Recovery
  - If the user has lost the password, they can request a recovery.
  - It starts by asking for an email address.
  - If the email address matches one of the known addresses, an email is sent to this address with a link.
  - If the link is clicked, the user can provide a new password. This new password is encrypted and replaces the old password.
  - From there, the user can retry login in since that they know the email and password.
1. Sign out
  - This is a simple operation that can be done by any signed in user.
1. Deleting credential
  - The user must be signed in
  - To delete credential, they are asked to enter their current password.
  - The password is encrypted and if it matches an email is sent with a warning message.
  - The credential is then tagged for deletion and the user signed out.
  - If the user signs in before a grace period, deletion is canceled and the credential returns to normal.
  - After the grace period a confirmation email is sent to the user and the credential is permanently deleted.
###Optional question/answer challenge
If this is implemented, the process is modified as follow:

1. Sign up
  - In addition to other information, the user can provide a question and the answer. The answer is encrypted.
  - The user can choose to not provide them, but this will prevent them from recovering if they loose their password. They can instead provide a meaningless challenge like foo/foo, in which case the challenge doesn't protect their credential.
1. Changing credential
  - The question/answer challenge can be changed like anything else   
  - The user is asked to enter their current password and the new question and answer.
  - The password is encrypted (as well as the new answer) and if it matches the current credential the change is applied.
1. Recovery
  - After clicking the link sent in the email, in addition to a new password the user can see the question and must provide an answer.
  - The answer is encrypted, and if it matches then the new password -also encrypted- replaces the old one.
1. Deleting credential
  - Requesting to complete the question/answer challenge doesn't make sense in this context. If a user knows the password, they can just change the question and answer, or remove them altogether, before deleting the credential. 
   
##The guide

###Setting up
To get the system ready, we are going to execute the following steps once.

+ Obtain a unique ID (UUID).<br>
This UUID is unique to the system to make sure that if several systems are implemented using this guide, they don't step on each other's toes. It doesn't provide any protection against attacks, because the UUID is not secret. But it helps avoid a user of one system being accidentally able to sign in to the other.<br>
To obtain a UUID, use for example the Tools/Create GUID menu in Microsoft Visual Studio. Any other mean of obtaining one is acceptable, all that is asked is that it's reasonably unique. For the rest of the document, let's assume the UUID is 16 bytes of binary data.

+ Choose a password hashing function (PHF). Here, I use Argon2d, because:
  * Argon2 is the winer of a [recent competition](https://password-hashing.net/) to find good functions.
  * There are many public implementations.
  * The 'd' variation, even though not the recommended one for password hashing, satisfies my priority number one better than others.<br>
If you choose another PHF, I ask it meets the following requirements:
    * You obtain the hash from 4 separate sources: password, UUID, salt (more on this later), additional data.
    * You can save and reuse the function settings, for example the size of the hash (settings can be empty).
- Choose a typical length.<br>
This length is the strength of your encryption. However, if an attacker just focus on finding the password using brute force, a length much larger than the average user's password length does not offer more protection. Because of this, the length just need to be *sufficient relatively to passwords*. Authors of Argon2 [recommend](https://github.com/P-H-C/phc-winner-argon2/blob/master/argon2-specs.pdf) (section 9) using 128 bits (16 bytes) of data.
- Choose additional data.<br>
This will depend of the complexity of your system, and how many different credential it's supposed to hold. The additional data piece is here to specify that the PHF encodes a password dedicated to user sign in. For the rest of this document, I'll assume it's the simple "password" string. If you implement the optional question/answer challenge, you can choose "answer" for the answer.
- Choose an transaction validity duration.<br>
This is how long an transaction is valid before a user must retry if they have not completed it. A typical duration is one day.

Setting up the software means some requirements must be fullfilled:

- The client must be able to execute custom code in some reasonably safe environment. For a web browser, for instance, that means enabling JavaScript or other technologies. We will assume the client is not subject to attacks trying to capture the password as the user is entering it, such as keyloggers. They are outside the scope of this document. If this could be a concern then the choice of the 'd' variation of Argon2 must be re-examined.
- The server is communicating with the client through a secure channel, for example using the HTTPS protocol. If this is not the case, and an attacker can eavesdrop on the communication, or play man-in-the-middle, they can easily fake a user and sign in to the system without knowing the password. However, priority number one is preserved, and they will not know what the password is.
- The server software is reasonably secure, meaning that it has the same level of protection than the communication channel mentioned above: if compromised, an attacker could could sign in, but they won't be able to get the password.
- The dabase is secure, for example because it's located on dedicated hardware in a safe place. The channel of communication between the server and the database (if not operating on the same hardware) must also be secure. Note that even if the database is compromised and an attacker obtains the list of users with their password, they will have the hardest time exploiting it, as this information is encrypted and cannot be reused on other systems.

The database will be prepared to hold the following information:

- The credentials table, recording
  - The username (note: set as unique, if the database allows it)
  - The email address (note: set as unique, if the database allows it)
  - The salt
  - The encrypted password
  - The password settings
  - The *active* flag (`false` means the credential is not valid, the term *active* comes from the concept of account activation)
  - A transaction code field
  - A transaction end field
  - A delete date field
  - The question (optional)
  - The encrypted secret answer (optional)
  - The answer settings (optional)
- The salts table, recording
  - A salt as binary data (note: set as unique, if the database allows it)
    
###Cleanup
In what follows, the server will sometimes "perform a database cleanup" step. This step is necessary, and executed at the specified time, only if the server doesn't use a separate cleanup thread. This is the case if for instance the server is completely implemented in PHP, because the server code is executed only upon request from the outside, and there is no thread.

If, on the other hand, the server is implemented in a threaded language such as C++ or Python, the cleanup can happen asynchronously and the "perform a database cleanup" step can be ignored.

The process of cleaning up the database is the following:

* Delete all entries in the credentials table that have the active field set to 0 and the current time greater than or equal to the transaction end field. This will free usernames and email addresses for which a sign up process was started but never completed.
* Delete all entries in the credentials table that have the active field set to 1 and the delete date field lesser than the current time. This will remove credentials that where scheduled for deletion. Send confirmation emails for these.
* Delete corresponding entries in the salts table to remove salts that are no longer used.
* Clear the transaction code field for all entries in the credentials table that have the active field set to 1 and the transaction_end field greater than or equal to the current time. This will prevent keeping transaction codes in the database longer than necessary.
  
###Terminology
In what follows, terms in bold indicate a specific operation:

- **Obtain from User** means showing a form, reading a hardware token, a file, and other means of obtaining that the user provide the requested information. 
- **Send to Server** is the operation, initiated by the client software, of sending a packet of data to the server. It can just be some data, or it can be a request to get more information from the server. In the later case, the server will respond with a **Return to Client** operation result.

Terms in italic indicate a specific piece of information. For example, *username*, or *password*. Note that *password* and *encrypted password* are two different things: the former is a sentence, the later some binary data.
When these terms are used in pseudo-code, they appear between <> signs. For example, `<username>`.

When terms are within parenthesis and separated with semicolons, it means they are sent in the same packet, or obtained at the same time (typically, from the same form). 
 
For all steps implying an operation on the database, whether a read or a write, I provide a sample pseudo-code query in SQL.

###Sign up
1. Client: **obtain from User** (*username*;*email address*). (For example, from a web form. There can be more information of course, but these are outside the scope of this document)
1. Client: **send to Server** (*username*;*email address*). 
1. Server: Perform a database cleanup.
1. Server: check if either *username* or *email address* is already used. (For example, perfom a `SELECT COUNT(username) FROM credentials WHERE username='<username>' OR email_address='<email address>';` query.
1. Server: if either is already used, **return to Client** an error.
1. Server: otherwise obtain a *salt*: cryptographically secure random binary data of the chosen length (in the Setting up section).
Obtaining a salt is implementation dependent. For example, in PHP you could use the `openssl_random_pseudo_bytes()` function.
Once you have a candidate salt, you can check for its uniqueness in the database, for example with a `INSERT INTO salts SET salt VALUES (<candidate>) ON DUPLICATE KEY UPDATE salt=salt;` query. This will both check for uniqueness and save it, turning this into an atomic operation.<br>
<br>
Repeat until the server has obtained a unique *salt*.<br>
<br>
1. Server: **return to client** (*salt*).
1. Client: **obtain from User** (*password*), or instead, optionally, (*password*;*question*;*answer*).
  - Note that at this stage asking the user to confirm their password is not needed, because they will eventually have to enter it again at a later stage.
  - For the same reason, if the optional question/answer challenge is implemented, confirming the answer is not needed as well.
  - The username and password can be obtained from the same form, but ideally the client software should ask for the password separately, since it reduces opportunities for an attacker to read it.
1. Client: use the password hashing function (PHF) to hash the password and create *encrypted password*, for example with `<encrypted password> = Argon2d(<password>, <salt>, UUID, Additional Data, <password settings>);`. Remember that additional data has been decided during the initial setting up phase. The rationale is as follow:
  - Using a unique salt, we ensure that all encrypted passwords are unique, even if two users enter the same password.
  - Using a cryptographically secure random salt of sufficient size ensures the password cannot be reconstructed from the hash, providing we trust Argon2 and the source of the salt.
  - Using a UUID ensures encrypted passwords are unique even between systems that don't know about each other (and therefore cannot check uniqueness of the salt).
  - Using additional data to indicate this is a password hash, we ensure that if the system uses the same salt and UUID for other purposes, the resulting encrypted hash will be different than the encrypted password.
1. Client: optionally use the PHF to hash the answer ans create *encrypted answer*, for example with `<encrypted answer> = Argon2d(<password>, <salt>, UUID, Additional Data For Answer, <answer settings>);`. Note that additional data for the answer must be different than for the password.
1. Client: **send to Server** (*username*;*email address*;*salt*;*encrypted password*;*password settings*) or instead, optionally (*username*;*email address*;*salt*;*encrypted password*;*password settings*;question;*encrypted answer*;*answer settings*).
1. Server: Create a *transaction code* with the chosen validity duration, and calculate the *transaction end* date at which the sign up request is rejected.
  - The transaction code must sufficiently unpredictable that an attacker cannot generate fake codes. If they could, they would be able to sign up with a fake address they don't control, then generate a link and try to activate the account with it. As long as a transaction code cannot be generated by attackers, the system is safe.
  - For this purpose, it is enough to use random data and mix it with the username, which is a known unique value (assuming the operation described in the next step is successful, but that's the point).
  - The suggested approach is create a string of N digits obtained from a random number generator (for example `openssl_random_pseudo_bytes()` in the case of PHP) where N is the length chosen during setup, append the username to it and hash the result. Since we are using Argon2 for password hashing, a logical choice is BLAKE2.  
1. Server: add credential information, the *transaction code* and *transaction end* to the credential database. For example with a `INSERT INTO credentials(username, email_address, salt, encrypted_password, password_settings, active, transaction_code, transaction_end) VALUES (<username>, <email address>, <salt>, <encrypted password>, <password setting>, 0, <transaction code>, <transaction end>);` query (add *question*, *encrypted answer* and *answer settings* as appropriate). This query can fail if two users try to sign up with the same username (or email address, but that's unlikely) roughly at the same time.
1. Server: **return to Client** success or failure.
  - In case of failure, the client could just apologize, repeat the process starting from step 1, and then tell one of the users that their username is now taken.
  - In case of success, display a message telling the user an email has been sent with a link, and clicking the link is required to finish signing up. Also tell the user the link is valid for a limited time. 
1. Server: in case of success, send an email to *email address* with some welcome message and a link to some activation page.
  - The link has two purposes: start the client software, and provide *transaction code* and *username* to the client. How this is performed varies widely between implementation, but in what follow we will assume *transaction code* is now known to the client, and that we now have a proof that the user controls the provided email address.
  - Specifically, with the link the client software must be able to run knowing *username*, *transaction code*, *salt* and *password settings* (and, optionally, *answer settings* as well).
  - Note that since the process has no control over how many times the user clicks the link, if they do it more than once there should be some ways to tell them the link is longer valid.
1. Client: **obtain from User** (*password*), or instead, optionally, display the question part of the challenge and obtain (*password*;*answer*).
1. Client: use the PHF to hash the password and create *encrypted password*. Optionally, also create *encrypted answer*.
1. Client: **send to Server** (*username*;*transaction code*;*encrypted password*;*password settings*) or (*username*;*transaction code*;*encrypted password*;*encrypted answer*;*answer settings*).
1. Server: Perform a database cleanup.
1. Server: complete the signup by matching the request with the previous record in the database. The provided information has the following requirements:
  - *username*, *transaction code*, *encrypted password*, *password settings* and optionally *encrypted answer* and *answer settings* must match.
  - The active flag must be initially 0, to prevent activating more than once.
  - The current time must be lesser than the *transaction end* date.
  - A successful activation must erase *transaction code* and *transaction end* from the database:  We don't want to keep outdated transaction codes.
  - Example of a database operation: `UPDATE credentials SET active = 1, transaction_code = NULL, transaction_end = NULL WHERE username = '<username>' AND password = '<encrypted_password>' AND password_settings = '<password settings>' AND active = 0 AND transaction_code = '<transaction code>' AND NOW() < transaction_end;`
  - Same example with the optional question/answer challenge implemented: `UPDATE credentials SET active = 1, transaction_code = NULL, transaction_end = NULL WHERE username = '<username>' AND password = '<encrypted_password>' AND password_settings = '<password settings>' AND answer = '<encrypted answer>' AND answer_settings = '<answer settings>' AND active = 0 AND transaction_code = '<transaction code>' AND NOW() < transaction_end;`
1. Server: **return to Client** success or failure.
1. Client: in case of success, notify the user of their successful sign up. In case of failure, there are several possibilities, like a password or answer that doesn't match, but also a credential already activated, or the validity period elapsed. Identifying the reason and being able to help the user fix it depends on the implementation, and won't be specified here. 

###Sign in

1. Client: **obtain from User** (*identifier*). The identifier can be either the username or email address.
1. Client: **send to Server** (*identifier*). 
1. Server: perform a database cleanup.
1. Server: check if *identifier* is a match for any active credential, by username or email address. If it is, get the associated *salt* and *password settings*. (For example, perfom a `SELECT salt, password_settings FROM credentials WHERE active=1 AND (username='<identifier>' OR email_address='<identifier>');` query).
1. Server: **return to client** either success and (*salt*;*password settings*), or failure.
1. Client: in case of failure, return to the first step.
1. Client: otherwise, **obtain from User** (*password*).
  - Similarly to sign up, the username and password could be in the same form, but ideally the client software asks for the password separately. 
2. Client: encrypt the password, using the user-provided value and server-provided salt and settings (see step 9 of sign up).
1. Client: **send to Server** (*identifier*;*encrypted password*;*password settings*). 
1. Server: check if the credential is a match for any active credential, by username or email address. If it is, grant access. (For example, perfom a `SELECT username, salt, password_settings, ... FROM credential WHERE active=1 AND password='<encrypted password>' AND password_settings='<password settings>' AND (username='<identifier>' OR email_address='<identifier>');` query).
  - How access is granted is not specified here, it depends on how the system will proceed from there. Typically, the server would generate a session ID that allows to read/write other databases.  
  - In the example query above, note the `, ...` ellipsis. It means "...and other fields", but it does **NOT** mean any of the encrypted fields, such as encrypted_password. These must never be selected. In particular, do **NOT** use a query starting with `SELECT * FROM credential WHERE ...`. The reason is that it could leak the password, something we can always avoid.
  - *Username*, *salt* and *password settings* are explicity selected so they can be returned to the client.
1. Server: in case of success, cancel any scheduled deletion, for example with a `UPDATE credentials SET delete_date = NULL WHERE username = '<username>' AND password = '<encrypted password>' AND password_settings = '<password settings>' AND active = 1 AND delete_date <> NULL;` query.
1. Server: **return to client** either success or failure, with in case of success the *username*, *salt*, *password settings* and whatever is necessary to continue using the system as a signed in user. If the delete date field was cleared in the database, it might be a good idea to also tell the client, to notify the user that their scheduled delete account operation has been canceled.   

###Change password

For all change of credential information, it is expected that the user is already signed in. However, it's a common practice to allow users to stay signed in over sessions by allowing them to keep the session token. Even if it was required from users to explicitely sign in every time they want to work with the system, they could still take a short break and leave their computer accessible by others with while the session is active.

For these reasons, it must be required that the user enter their password any time they want to change part of their credential information: username, password, or email address.

This section is specific to changing the password, but considerations stated above are also valid for other changes.

1. Client: **obtain from User** (*password*;*new password*).
  - Since there will be no other opportunities for the user to confirm the new password, it's a good idea to ask them to type it twice at this stage, and verify the two versions are equal.
  - Because we require the user to be already signed in, *username*, *salt* and *password settings* are known to the client software.
1. Client: encrypt the old and new passwords, using the user-provided values (see step 9 of sign up).
  - This section doesn't cover it, but if for some reason password settings had to be changed, it would be the right time to do it. In what follows, we will therefore consider *password settings* and *new password settings* to be different. In practice, the client software will just reuse old settings for the new password. 
1. Client: **send to Server** (*username*;*encrypted password*;*password settings*;*encrypted new password*;*new password settings*). 
1. Server: perform a conditional replacement of the old password and old password settings with the new ones.
  - The *username*, *encrypted password* and *password settings* must obviously match what's in the database.
  - The credential must be active.
  - This is done with a query similar to `UPDATE credentials SET password = '<encrypted new password>', password_settings = '<new password settings>' WHERE username = '<username>' AND password = '<encrypted password>' AND password_settings = '<password settings>' AND active = 1;`.
1. Server: **return to client** success or failure.
  - Failure can be, for example, if the current password doesn't match what's in the database.

###Change email address

See the [change password](#change_password) section above for considerations regarding changing credential information.

1. Client: **obtain from User** (*password*;*new email address*).
  - A confirmation email will be sent to the new address. Therefore, it is not necessary to ask the user to confirm the address in other ways, such as entering it twice.
  - Because we require the user to be already signed in, *username*, *salt* and *password settings* are known to the client software.
1. Client: encrypt the password, using the user-provided value (see step 9 of sign up).
1. Client: **send to Server** (*username*;*encrypted password*;*password settings*;*new email address*). 
1. Server: perform a conditional replacement of the email address with the new one.
  - The *username*, *encrypted password* and *password settings* must obviously match what's in the database.
  - The credential must be active.
  - This is done with a query similar to `UPDATE credentials SET email_address = '<new email address>' WHERE username = '<username>' AND password = '<encrypted password>' AND password_settings = '<password settings>' AND active = 1;`.
1. Server: in case of success, send a confirmation email at the new address.
1. Server: **return to client** success or failure.
  - Failure can be, for example, if the current password doesn't match what's in the database.
  - Note that we don't check at this time if the address is valid. This is because the email must go through actors that we don't control, such as email servers, spam filters and other niceties of the internet. Therefore, the client at this point should display message that looks like this:
> Check your reception box for the confirmation email. If you did not receive it, it can be due to several causes, for example a spam filter, but one of them is that you may have misspelled the address. The change won't be complete until you see the confirmation email.

###Change username

See the [change password](#change_password) section above for considerations regarding changing credential information.

1. Client: **obtain from User** (*password*;*new username*).
  - Since the new username can be displayed in plain text and will take effect immediately, there is no need to confirm it, for example by entering it twice.
  - Because we require the user to be already signed in, *username*, *salt* and *password settings* are known to the client software.
1. Client: encrypt the password, using the user-provided value (see step 9 of sign up).
1. Client: **send to Server** (*username*;*encrypted password*;*password settings*;*new username*). 
1. Server: perform a conditional replacement of the username with the new one.
  - The *username*, *encrypted password* and *password settings* must obviously match what's in the database.
  - The credential must be active.
  - This is done with a query similar to `UPDATE credentials SET username = '<new username>' WHERE username = '<username>' AND password = '<encrypted password>' AND password_settings = '<password settings>' AND active = 1;`.
1. Server: **return to client** success or failure.
  - Failure can be, for example, if the current password doesn't match what's in the database.
  - Failure can also be that the new username is taken. The server should, if possible, return a specific error for this situation so the client software can display the appropriate message.

###Change question/answer challenge (optional)

See the [change password](#change_password) section above for considerations regarding changing credential information.
This section is optional and obviously applies only if the question/answer chanllenge is implemented. 

1. Client: **obtain from User** (*password*;*new question*;*new answer*).
  - Since there will be no other opportunities for the user to confirm the new answer, it's a good idea to ask them to type it twice at this stage, and verify the two versions are equal.
  - Because we require the user to be already signed in, *username*, *salt* and *password settings* are known to the client software.
1. Client: encrypt the password, using the user-provided values (see step 9 of sign up).
1. Client: **send to Server** (*username*;*encrypted password*;*password settings*;*new question*;*encrypted new answer*;*new answer settings*). 
1. Server: perform a conditional replacement of the question and answer with the new ones.
  - The *username*, *encrypted password* and *password settings* must obviously match what's in the database.
  - The credential must be active.
  - This is done with a query similar to `UPDATE credentials SET question = '<new question>', answer = '<encrypted new answer>', answer_settings = '<new answer settings>' WHERE username = '<username>' AND password = '<encrypted password>' AND password_settings = '<password settings>' AND active = 1;`.
1. Server: **return to client** success or failure.
  - Failure can be, for example, if the current password doesn't match what's in the database.

###Recovery

Recovering a credential for which the password has been lost or forgotten is done using the email address. The user, by showing that they control the email address, can change the associated password and sign in using the address and the changed password.

If the optional question/answer challenge is implemented, they will also have to answer the question in the credential associated to the email address. The new password will be applied only if the answer is valid.
  
1. Client: **obtain from User** (*email address*). Note that the user isn't signed in at this time, and the client software knows nothing about them.
1. Client: **send to Server** (*email address*).
1. Server: Create an *transaction code* with the chosen validity duration, and calculate the *transaction end* date at which the recovery request is rejected. See step 12 of the sign up section for details about the transaction code.
1. Server: update the *transaction code* and *transaction end* date in the credential database. For example with a `UPDATE credentials SET transaction_code = '<transaction_code>', transaction_end = '<transaction_end>' WHERE email_address = '<email address>' AND active = 1 AND (NOT (question IS NULL));` query.
  - This query can fail if either the email address doesn't exist, or the question is empty.
  - If the question is empty, it means the user explicitely denied the possibility of recovery.
1. Server: **return to Client** success or failure.
1. Client: display the result.
  - In case of failure, the client should be able to distinguish between a bad email address or an empty question, and display the corresponding message.
  - In case of success, display a message telling the user an email has been sent with a link, and clicking the link is required to finish recovering. Also tell the user the link is valid for a limited time. 
1. Server: in case of success, send an email to *email address* with some warning message and a link to some recovery page.
  - The link has two purposes: start the client software, and provide *transaction code*, *username*, *salt*, *question* and *answer settings* to the client. How this is performed varies widely between implementation, but in what follow we will assume *transaction code* is now known to the client, and that we now have a proof that the user controls the provided email address.
  - Note that since the process has no control over how many times the user clicks the link, if they do it more than once there should be some ways to tell them the link is longer valid.
1. Client: display the question, ask for the answer and a new password.
1. Client: **obtain from User** (*answer*;*new password*).
1. Client: use the PHF to hash the answer and new password and create *encrypted answer* and *encrypted new password*.
1. Client: **send to Server** (*username*;*transaction code*;*encrypted answer*;*answer settings*;*encrypted new password*;*new password settings*).
1. Server: perform a database cleanup.
1. Server: complete the recovery by looking for a match of the request in the database. The provided information has the following requirements:
  - *username*, *transaction code*, *encrypted answer* and *answer settings* must match.
  - The active flag must be 1.
  - The current time must be lesser than the *transaction end* date.
  - A successful recovery must erase *transaction code* and *transaction end* from the database.
  - The delete date field should also be cleared, to avoid the corner case where a recovery and deletion happen at the same time. 
  - Example of a database operation: `UPDATE credentials SET password = '<encrypted new password>', password_settings = '<new password settings>', transaction_code = NULL, transaction_end = NULL, delete_date = NULL WHERE username = '<username>' AND answer = '<encrypted answer>' AND answer_settings = '<answer settings>' AND active = 1 AND (NOT (question is NULL)) AND transaction_code = '<transaction_code>' AND (NOW() < transaction_end);`
1. Server: **return to Client** success or failure.
1. Client: in case of success, notify the user of their successful recovery. In case of failure, there are several possibilities, like an answer that doesn't match, but also a credential not activated yet, activated more than once, or the validity period elapsed. Identifying the reason and being able to help the user fix it depends on the implementation, and won't be specified here. 

###Sign out

There is nothing that can specified for this operation, it completely depends on the implementation.

###Deleting credentials

As noted in the [overview](#overview), deleting the account isn't instantaneous. This is not to prevent an attacker having gained temporary control of the credential from being able to delete it. A password (at least) is required to do it, and if the attacker got the password they can do whatever they want. The delay exists as a last-resort safety, in case the user changes their mind before it becomes effective.

1. Client: **obtain from User** (*password*).
  - Because we require the user to be already signed in, *username*, *salt* and *password settings* are known to the client software.
1. Client: encrypt the password using the user-provided value (see step 9 of sign up).
1. Client: **send to Server** (*username*;*encrypted password*;*password settings*). 
1. Server: read the email address corresponding to this user, and tag the credential for deletion.
  - The *username*, *encrypted password* and *password settings* must obviously match what's in the database.
  - The credential must be active.
  - The email address can be obtained for example with a `SELECT email_address WHERE username = '<username>' AND password = '<encrypted password>' AND password_settings = '<password settings>' AND active = 1;` query.
  - Tagging the credential for deletion can be done for example with a `UPDATE credentials SET delete_date = <delete date> WHERE username = '<username>' AND password = '<encrypted password>' AND password_settings = '<password settings>' AND active = 1;` query.
1. Server: in case of success, send a warning message at the email address.
1. Server: **return to client** success or failure.
  - Failure can be, for example, if the current password doesn't match what's in the database.
1. Server: perform all operations corresponding to a sign out. 
1. Client: perform all operations corresponding to a sign out. 
1. Client: display the result.
  - In case of failure, the reason.
  - In case of success, tell the user that they have been signed out, and that if they sign in again at any time during the grace period, deleting the account will be automatically canceled.
