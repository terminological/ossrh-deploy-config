# ossrh-deploy-config

Deploy a artefact to OSSRH

edit the file `~/.m2/settings.xml` to look like the following:

```XML
<settings>
  <servers>
    <server>
      <id>ossrh</id>
      <username>my-jira-username</username>
      <password>my-jira-password</password>
    </server>
  </servers>
  <profiles>
    <profile>
      <id>ossrh</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <properties>
        <gpg.executable>gpg2</gpg.executable>
        <gpg.passphrase>my-gpg-passphrase</gpg.passphrase>
        <gpg.keyname>my-gpg-keyname</gpg.keyname>
      </properties>
    </profile>
  </profiles>
</settings>

```

`my-jira-username` and `my-jira-password` were set up when signing up to sonatype
Set up a gpg if there is not one already:

```BASH
sudo apt-get install pinentry-tty
export GPG_TTY=`tty`
nano ~/.gnupg/gpg-agent.conf
```

and adding / changing the line:

```
pinentry-program /usr/bin/pinentry-tty
```

then reloading:

```BASH
gpg-connect-agent reloadagent /bye
```

N.B. I saw something about using a loopback instead of pinentry?
Also something that does not put the gpg passphrase in the settings.xml which I guess is 
a security risk involving a `passphraseServerId`.

## Generate a key using a real name and email:

```
gpg2 --gen-key

```

Copy the passphrase (as `my-gpg-passphrase`) into the `settings.xml`.
List the signatures:

```BASH
gpg2 --list-sigs
```

Which results in:

```
ub   rsa2048/XXXXXXXX 2022-06-15 [SC]
uid         [ultimate] Full name <full.name@example.com>
sig 3        XXXXXXXX 2022-06-15  Full name <full.name@example.com>
sub   rsa2048/YYYYYYYY 2022-06-15 [E]
sig          XXXXXXXX 2022-06-15  Full name <full.name@example.com>
```

XXXXXXXX is the Gpg key name we wish to use (`my-gpg-keyname` in `settings.xml`). Send it to the ubuntu key server.

```BASH
gpg --keyserver keyserver.ubuntu.com --send-keys XXXXXXXX
```

then test gpg is working:

```BASH
echo test | gpg2 -a --sign
```

## Deploy the project to upload a snapshot

```BASH
export GPG_TTY=`tty`
cd ~/Git/my-project
mvn clean deploy
```
