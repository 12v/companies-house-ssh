# SSH Test

## Background

This project tests (publicly!) whether a public key is configured correctly for a specific username on a specific server.  All sensitive information is obfuscated.  We configure a GitHub Actions action to attempt a SFTP connection using the appropriate keys, and log the results.


## Results

From the latest results [here](https://github.com/12v/ssh-test/actions/runs/6697895397/job/18198747787), you can see that:
 * The SHA256 of `<username>@<hostname>` is `1c0622b93f9569f314556aab2e373ff4f1314cb3de58f2c74a44bb077cd72594` (from [here](https://github.com/12v/ssh-test/actions/runs/6697895397/job/18198747787#step:5:9))
 * The SHA256 of the server's public key is `zLWu0B78gJqq315/f38HVSYoMUOv1utlqDQ0PZsfAFo` (from [here](https://github.com/12v/ssh-test/actions/runs/6697895397/job/18198747787#step:6:67))
 * The SHA256 of the client's public key is `y7sR7uUSfW9yy4riD8MpRdiFwXADSutMrk3o0M13t48` (from [here](https://github.com/12v/ssh-test/actions/runs/6697895397/job/18198747787#step:6:102))
 * The client can reach the server, but the server responds to the public key with `receive packet: type 51` (from [here](https://github.com/12v/ssh-test/actions/runs/6697895397/job/18198747787#step:6:93))

Packet type 51 corresponds to a `SSH_MSG_USERAUTH_FAILURE` response (defined [here](https://www.ietf.org/rfc/rfc4252.txt)), indicating that the server is rejecting the public key as not valid for the requested user.


## FAQs

### Does the client have connectivity to the server?
Yes, the client can reach the server - the error response is received from the SSH daemon running on the server.

### Is the username / hostname / public key correct?
Yes, this can be validated by comparing hashes.

### Is the private key correct?
Yes, OpenSSH won't get as far as attempting to use a public key if the corresponding private key is not present.

### Is there a client-side environment issue preventing this from working?
Perhaps... but this result is consistent with what's seen in other environments/ISPs/OSs/etc.


## Security

The following are configured as GitHub Actions secrets:
* The private key (because this is a secret)
* The username and hostname (because this is potentially sensitive)

Nonetheless, the username and hostname we are using can be validated by comparing the calculated SHA256 hash with a SHA256 hash calculated locally as follows:

`echo -n "<username>@<hostname>" | shasum -a 256`

If we are using the username and hostname that you have in mind, the SHA256 should be `1c0622b93f9569f314556aab2e373ff4f1314cb3de58f2c74a44bb077cd72594` (from [here](https://github.com/12v/ssh-test/actions/runs/6697895397/job/18198747787#step:5:9)).
