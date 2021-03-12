# Email OAuth 2.0 Proxy
An IMAP/SMTP proxy that adds OAuth 2.0 authentication transparently for clients that don't support this method.


## Motivation and alternatives
Many email services that provide IMAP/SMTP access require OAuth 2.0 to authenticate the connection, but not all native clients support this. This Python script is a simple local proxy that intercepts authentication commands and transparently replaces them with the appropriate OAuth 2.0 commands and credentials.

[DavMail](http://davmail.sourceforge.net/) is an alternative that takes the same approach of providing a local IMAP/SMTP server (and more) for Exchange/Office 365, though it does this by translating these protocols into Exchange API calls rather than proxying the connection. That approach is very useful in situations where IMAP/SMTP are not supported, but it has limitations in terms of speed and the number of messages that can be retrieved. This proxy was developed to work around some of these limitations for providers that do support IMAP/SMTP natively.


## Getting started
First, add configuration details for each server and account you want to use with the proxy in the file `emailproxy.config`. [Sample account configurations](emailproxy.config) are provided for Office 365 and Gmail, but you will need to register a new [Microsoft identity](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app) or [Google API](https://support.google.com/googleapi/answer/6158849) client to get started (or use an existing client ID and secret).

Example server configurations are provided for Office 365 and Gmail, but other services that use OAuth 2.0 should also work (please [open an issue](https://github.com/simonrob/email-oauth2-proxy/issues) if not). You can remove details for services you don't use, or add additional ones for any other OAuth 2.0 IMAP/SMTP servers you would like to use with the proxy. Multiple accounts with the same service can share the same server, and the correct server to use with an account is identified using the port number you configure in your client (see below).

Next, install the script's requirements: `pip3 install -r requirements.txt`, and start the proxy: `python3 emailproxy.py` – a menu bar icon should appear.

Finally, open your email client and configure its server details to match those set in the configuration file. For example, using the sample Office 365 details, this would be `localhost` on port `1433` for IMAP and `localhost` on port `1587` for SMTP. The local connection in your email client should be configured as unencrypted, but the connection between the proxy and your email server is secured (SSL for IMAP, SSL/STARTTLS for SMTP).

The first time your email client makes a request you should see a notification from the proxy about authentication. Click the `Authorise account...` option from the menu bar icon and follow the instructions to log in to your account. After authentication completes you should have IMAP/SMTP access to your account as normal.


## Running as a service/daemon (macOS)
Move [the included plist file](ac.robinson.email-oauth2-proxy.plist) to `~/Library/LaunchAgents/`, then edit it to replace `/path/to/python3` and `/path/to/emailproxy.py` with the full paths to your local `python3` installation and the `emailproxy.py` script. Run `launchctl load ~/Library/LaunchAgents/ac.robinson.email-oauth2-proxy.plist`, and the menu bar icon should appear. The script will now run at startup.

If you stop the service (i.e., `Quit` in the menu bar), you can restart it using `launchctl start ac.robinson.email-oauth2-proxy`. You can remove the service from your startup items using `launchctl unload ~/Library/LaunchAgents/ac.robinson.email-oauth2-proxy.plist`.


## Troubleshooting
If you encounter problems, enabling `Debug mode` from the menu will send all client-proxy-server communication to the system log to help identify the error. On macOS this can be viewed using Console.app (select `system.log` in the sidebar).

Please note that Debug mode may also result in your login credentials being printed to the log (though this is avoided where possible). However, it is worth pointing out that while the username you set in your email client must be correct, the password used does not need to be the one you actually use to log in to your account, so you can use a test password for debugging and then replace this with a secure password (and authenticate again) once set up.

Please feel free to [open an issue](https://github.com/simonrob/email-oauth2-proxy/issues) reporting any bugs you find, or [submit a pull request](https://github.com/simonrob/email-oauth2-proxy/pulls) to help improve this tool.


## Potential improvements and known issues ([pull requests](https://github.com/simonrob/email-oauth2-proxy/pulls) welcome)
- Improved error handling (e.g., totally reliable recovery from exceptions during computer sleep; robustness improvements)
- Testing on different platforms and with different providers (currently tested only with Office 365 and Gmail on macOS)
- Authentication currently relies on [pywebview](https://github.com/r0x0r/pywebview/) to display the account login page. For reasons that are currently not clear, the system component that pywebview uses can get into a state where the local login completion redirection URL does not load (pywebview simply hangs). Restarting seems to be the only reliable fix for this
- Encrypted local connections?
- Package as .app/.exe etc?


## License
Apache 2.0
