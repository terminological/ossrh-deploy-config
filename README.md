# ossrh-deploy-config

Deploy a artefact to OSSRH as part of the terminological organisation

edit the file `~/.m2/settings.xml` to look like the following:

```XML
<settings>
  <servers>
    <server>
      <id>ossrh</id>
      <username>my-jira-username</username>
      <password>my-jira-password</password>
    </server>
    <server>
      <id>my-gpg-keyname</id>
      <passphrase>my-gpg-passphrase</passphrase>
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

## Setup project to be deployed

Setup the `my-git-repo/pom.xml` to use this project as a parent and reconfigure scm location to point at `my-git-repo`

```XML
  ...
  <parent>
    <groupId>io.github.terminological</groupId>
    <artifactId>ossrh-deploy-config</artifactId>
    <version>0.3.0-SNAPSHOT</version>
  </parent>
  ...
  <properties>
    <git.repo>my-git-repo</git.repo>
  </properties>
  ...
  <!-- might not be required as in parent pom
  <scm>
    <connection>scm:git:git@github.com:terminological/${git.repo}.git</connection>
    <developerConnection>scm:git:git@github.com:terminological/${git.repo}.git</developerConnection>
    <url>https://github.com/terminological/${git.repo}</url>
  </scm>
  -->
  ...
  
```
## Deploy the my-git-repo project to upload a main-SNAPSHOT

```BASH
cd ~/Git/my-project
mvn versions:set -DnewVersion=main-SNAPSHOT
mvn clean deploy
```

## Deploy a release to Maven central

```BASH

cd ~/Git/my-project

git add -A
git commit -m "pre release-X.Y.Z"
git push

git checkout -b release-X.Y.Z
mvn versions:set -DnewVersion=X.Y.Z

mvn clean deploy
# fix any issues and repeat deploy
git add -A
git commit -m "post release-X.Y.Z"
git push --set-upstream origin release-X.Y.Z
gh release create X.Y.Z --target=release-X.Y.Z --generate-notes

git checkout main

# may have to consider
git merge release-X.Y.Z
mvn versions:set -DnewVersion=main-SNAPSHOT

# if everything goes wrong
git branch -d release-X.Y.Z
git push origin --delete release-X.Y.Z

```

TODO: I should understand how this fits in:

https://central.sonatype.org/publish/publish-maven/#performing-a-release-deployment
