# Authorizer

* WordPress Plugin: [https://wordpress.org/plugins/authorizer/][wp]
* Changelog: [https://github.com/uhm-coe/authorizer/blob/master/readme.txt][changelog]

*Authorizer* is a WordPress plugin that restricts access to specific users, typically students enrolled in a university course. It maintains a list of approved users that you can edit to determine who has access. It also replaces the default WordPress login/authorization system with one relying on an external server, such as Google, CAS, or LDAP. Finally, *Authorizer* lets you limit invalid login attempts to prevent bots from compromising your users' accounts.

*Authorizer* requires the following:

* **CAS server** or **LDAP server** (plugin needs the URL)
* PHP extensions: php5-mcrypt, php5-ldap, php5-curl

*Authorizer* provides the following options:

* **Authentication**: WordPress accounts; Google accounts; CAS accounts; LDAP accounts
* **Login Access**: All authenticated users (all local and all external can log in); Only specific users (all local and approved external users can log in)
* **View Access**: Everyone (open access); Only logged in users
* **Limit Login Attempts**: Progressively increase the amount of time required between invalid login attempts.

## LDAP Role mapping

If you're willing to use filter hooks, there should be a way to do this now. I still plan on building out a UI for defining group mappings in the Dashboard settings page, but in the meantime:

There is a filter called "authorizer_custom_role" that lets you inspect attributes returned from CAS or LDAP when a user logs in, and change the user's role to anything you'd like. Here's an example of its use:

```
/**
 * Filter the default role of the currently logging in user based on any of
 * their user attributes.
 *
 * @param string $default_role Default role of the currently logging in user.
 * @param array $user_data     User data returned from external service.
 */
function my_authorizer_custom_role( $default_role, $user_data ) {
  // Allow library guests to log in via CAS, but only grant them 'subscriber' role.
  if (
    isset( $user_data['cas_attributes']['eduPersonPrimaryAffiliation'] ) &&
    'library-walk-in' === $user_data['cas_attributes']['eduPersonPrimaryAffiliation']
  ) {
    $default_role = 'subscriber';
  }
  return $default_role;
}
add_filter( 'authorizer_custom_role', 'my_authorizer_custom_role', 10, 2 );
```
If you also want to automatically approve users based on data from CAS or LDAP (useful if you have the "Who can log into this site?" option set to "Only approved users"), skipping the "Pending User" step, I'm implementing a new filter called "authorizer_automatically_approve_login" that is similar. Here's an example of its use:

```
/**
 * Filter whether to automatically approve the currently logging in user
 * based on any of their user attributes.
 *
 * @param bool  $automatically_approve_login
 *   Whether to automatically approve the currently logging in user.
 * @param array $user_data User data returned from external service.
 */
function approve_all_faculty_logins( $automatically_approve_login, $user_data ) {
  // Automatically approve logins for all faculty members.
  if (
    isset( $user_data['cas_attributes']['eduPersonAffiliation'] ) &&
    'faculty' === $user_data['cas_attributes']['eduPersonAffiliation']
  ) {
    $automatically_approve_login = true;
  }
  return $automatically_approve_login;
}
add_filter( 'authorizer_automatically_approve_login', 'approve_all_faculty_logins', 10, 2 );
Version 2.6.8 should have that last filter, and it is due out in a day or two.
```

## Screenshots

![](assets/screenshot-1.png?raw=true "WordPress Login screen with Google Logins and CAS Logins enabled.")
![](assets/screenshot-2.png?raw=true "Authorizer Dashboard Widget.")
![](assets/screenshot-3.png?raw=true "Authorizer Options: Access Lists.")

[wp]: https://wordpress.org/plugins/authorizer/
[changelog]: https://github.com/uhm-coe/authorizer/blob/master/readme.txt
