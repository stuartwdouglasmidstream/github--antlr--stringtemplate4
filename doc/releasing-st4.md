# Cutting an ST Release

## Github

Create a pre-release or full release at github. 

### Delete existing release tag

Wack any existing tag as mvn will create one and it fails if already there.

```
$ git tag -d 4.3.2
$ git push origin :refs/tags/4.3.2
$ git push upstream :refs/tags/4.3.2
```

## Bump version
 
Here is a simple script to display any line from the critical files with, say, `4.3.2` in it:

```bash
find . -type f -exec grep -l '4\.3\.2' {} \;
```

For sure change `ST.java`:

```java
public final static String VERSION = "4.3.2";
```

Commit to repository.

## Maven Repository Settings

First, make sure you have maven set up to communicate with staging servers etc...  Create file `~/.m2/settings.xml` with appropriate username/password for staging server and gpg.keyname/passphrase for signing. Make sure it has strict visibility privileges to just you. On unix, it looks like:

```bash
beast:~/.m2 $ ls -l settings.xml 
-rw-------  1 parrt  staff  914 Jul 15 14:42 settings.xml
```

Here is the file template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
  User-specific configuration for maven. Includes things that should not
  be distributed with the pom.xml file, such as developer identity, along with
  local settings, like proxy information.
-->
<settings>
   <servers>
        <server>
          <id>sonatype-nexus-staging</id>
          <username>sonatype-username</username>
          <password>XXX</password>
        </server>
        <server>
          <id>sonatype-nexus-snapshots</id>
          <username>sonatype-username</username>
          <password>XXX</password>
        </server>
   </servers>
    <profiles>
            <profile>
              <activation>
                    <activeByDefault>false</activeByDefault>
              </activation>
              <properties>
                    <gpg.keyname>UUU</gpg.keyname>
                    <gpg.passphrase>XXX</gpg.passphrase>
              </properties>
            </profile>
    </profiles>
</settings>
```

## Maven deploy snapshot

The goal is to get a snapshot, such as `4.3.2-SNAPSHOT`, to the [staging server](https://oss.sonatype.org/#stagingRepositories).

Do this:

```bash
$ mvn deploy -DskipTests
...
[INFO] --- maven-deploy-plugin:2.7:deploy (default-deploy) @ ST4 ---
Downloading from sonatype-nexus-snapshots: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/ST4/4.3.2-SNAPSHOT/maven-metadata.xml
Downloaded from sonatype-nexus-snapshots: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/ST4/4.3.2-SNAPSHOT/maven-metadata.xml (756 B at 925 B/s)
Uploading to sonatype-nexus-snapshots: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/ST4/4.3.2-SNAPSHOT/ST4-4.3.2-20181110.190125-4.jar
Uploaded to sonatype-nexus-snapshots: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/ST4/4.3.2-SNAPSHOT/ST4-4.3.2-20181110.190125-4.jar (302 kB at 257 kB/s)
Uploading to sonatype-nexus-snapshots: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/ST4/4.3.2-SNAPSHOT/ST4-4.3.2-20181110.190125-4.pom
Uploaded to sonatype-nexus-snapshots: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/ST4/4.3.2-SNAPSHOT/ST4-4.3.2-20181110.190125-4.pom (2.6 kB at 4.3.2 kB/s)
Downloading from sonatype-nexus-snapshots: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/ST4/maven-metadata.xml
Downloaded from sonatype-nexus-snapshots: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/ST4/maven-metadata.xml (370 B at 1.7 kB/s)
Uploading to sonatype-nexus-snapshots: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/ST4/4.3.2-SNAPSHOT/maven-metadata.xml
Uploaded to sonatype-nexus-snapshots: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/ST4/4.3.2-SNAPSHOT/maven-metadata.xml (756 B at 1.2 kB/s)
Uploading to sonatype-nexus-snapshots: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/ST4/maven-metadata.xml
Uploaded to sonatype-nexus-snapshots: https://oss.sonatype.org/content/repositories/snapshots/org/antlr/ST4/maven-metadata.xml (370 B at 599 B/s)
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  8.672 s
[INFO] Finished at: 2018-11-10T11:01:28-08:00
[INFO] ------------------------------------------------------------------------
```

## Maven release

The maven deploy lifecycle phased deploys the artifacts and the poms for the ANTLR project to the [sonatype remote staging server](https://oss.sonatype.org/content/repositories/snapshots/org/antlr/ST4).

```bash
mvn deploy -DskipTests
```

Now, do this:

```bash
mvn release:prepare -Darguments="-DskipTests"
```

Hm...per https://github.com/keybase/keybase-issues/issues/1712 we need this to make gpg work (needed for releasing not build):

```bash
export GPG_TTY=$(tty)
```

It will start out by asking you the version number:

```
...
What is the release version for "StringTemplate 4"? (org.antlr:ST4) 4.3.2: : 
What is SCM release tag or label for "StringTemplate 4"? (org.antlr:ST4) ST4-4.3.2: : 4.3.2           
What is the new development version for "StringTemplate 4"? (org.antlr:ST4) 4.3.3-SNAPSHOT: : 4.3.3-SNAPSHOT
...
```

Now release

```bash
mvn release:perform -Darguments="-DskipTests"
```

Maven will use git to push pom.xml changes.

Now, go here:

&nbsp;&nbsp;&nbsp;&nbsp;[https://oss.sonatype.org/#welcome](https://oss.sonatype.org/#welcome)

and on the left click "Staging Repositories". You click the staging repo and close it, then you refresh, click it and release it. It's done when you see it here:

&nbsp;&nbsp;&nbsp;&nbsp;[https://oss.sonatype.org/service/local/repositories/releases/content/org/antlr/ST4/4.3.2](https://oss.sonatype.org/service/local/repositories/releases/content/org/antlr/ST4/4.3.2)

All releases should be here: [https://repo1.maven.org/maven2/org/antlr/ST4/](https://repo1.maven.org/maven2/org/antlr/ST4/).

Seems to take a while to propogate.

## Javadoc

```bash
mvn javadoc:javadoc
```

```bash
cp -r ~/antlr/code/stringtemplate4/target/apidocs/* ~/antlr/sites/website-st4/api```

# Update website

Copy the jars to stringtemplate.org site and update download/index.html

```bash
cp ~/.m2/repository/org/antlr/ST4/4.3.2/ST4-4.3.2.jar ~/antlr/sites/website-st4/download/ST-4.3.2.jar
cd ~/antlr/sites/website-st4/download
git add ST-4.3.2.jar
```

## Update site

Find stuff:

```
cd ~/antlr/sites/website-st4
find . -type f -exec grep '4\.2' {} \;
vi index.html scripts/topnav.js download.html 
```

