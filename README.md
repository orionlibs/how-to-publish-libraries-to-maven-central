# how-to-publish-libraries-to-maven-central
The steps I followed to successfully publish my Java, Maven-based libraries to Maven Central
--------------------------------------------------------------------------------------------
1. Guide directly from Sonatype that manages Maven Central:  
http://central.sonatype.org/pages/ossrh-guide.html
2. Create account. Only needed once:  
https://issues.sonatype.org/secure/Signup!default.jspa
3. Create ticket to publish your first library, but this is also needed for them to setup your groupID. This setup for the groupID is needed once: 
https://issues.sonatype.org/secure/CreateIssue.jspa?pid=10134&issuetype=21  
Provide groupID. If you have a GitHub-based repo to publish like "my-repo" and this repo is under the GitHub username "my-username" then the groupID will be "io.github.my-username".  
The artifactID of your library, like, "my-repo".
4. Create a public empty repo in your account "my-username" with the name they will give you in the email they automatically send like: "OSSRH-93147" so that the link "https://github.io/my-username/OSSRH-93147" is valid. This is how they check that you own "my-repo". You can delete this repo after the registration of your groupID succeeds. They will email you about it.
5. Go to your ticket and make it "Open". They will take care of things in a day or 2. They will mark it as "Closed" and they will email you about it and tell you that the registration process is complete.
6. Install GnuPG (WinGPG for Windows users).
7. run the command "gpg --gen-key" to generate a GPG key which is used to sign your libraries files that will be published to Maven Central.
8. The key generated will be in the folder (for Windows)  
"C:/Users/my-username/.gnupg/openpgp-revocs.d"  
and the file will look like: 635K75FAB379825D68B78TP347H32HGYR3L6904.rev
9. Then you need to register this key with the GPG servers. The guide gives the following 3, but the second was the one that worked for me. Run one of the following commands:  
"gpg --keyserver keyserver.ubuntu.com --send-keys 635K75FAB379825D68B78TP347H32HGYR3L6904"  
OR  
"gpg --keyserver keys.openpgp.org --send-keys 635K75FAB379825D68B78TP347H32HGYR3L6904"  
OR  
"gpg --keyserver pgp.mit.edu --send-keys 635K75FAB379825D68B78TP347H32HGYR3L6904"
10. Then run "gpg --list-signatures --keyid-format 0xshort" to get the hexadecimal code of your key. In the output there is a line that says "sig3" and next to it there is a value that looks like: "0xB5D5824C". Keep that for step 18.
11. When your GPG key expires then run  
"gpg --edit-key 635K75FAB379825D68B78TP347H32HGYR3L6904"  
to reset its expiration date. When you run this command it will take you the GPG prompt that says "gpg>".  
Run "expire". Enter "2y", for example to make the key last for another 2 years.  
Then run "save".  
Then run one of the commands in step 9 to register your key again.
12. Create the file "C:/Users/my-username/.gnupg/gpg-agent.conf" with 2 lines in it:  
default-cache-ttl 34560000  
max-cache-ttl 34560000
13. Run "mvn clean install" in your project root folder from the command-line to save the GPG password so that it does not asks for it every time.
14. Run "gpgconf --kill gpg-agent"
15. Run "gpgconf --launch gpg-agent"
16. In your Maven folder in "C:/Users/my-username/.m2" there should be a file called settings.xml or if there is one and you don't want to change it, just create another one like "settings-maven-central.xml".
17. Inside this file put the following:  
```xml
<settings>
  <servers>
    <server>
      <id>ossrh</id>
      <username>my-github-username</username>
      <password>my-sonatype-JIRA-password</password>
    </server>
  </servers>
</settings>
```
18. There are some Maven plugins you need to add to your project pom.xml in order for the artifacts to be signed and so that you can deploy from your command line. I could not make "mvn clean install" work in my IntelliJ, because of the GPG key and I couldn't find anything online to help me fix it. So, I run this command and the deployment command form the command-line, which is fine for me. These are the plugins and the other config:  
```xml
<scm>
    <url>https://github.com/orionlibs/spring-http-request-logger</url>
    <connection>scm:git:https://github.com/orionlibs/spring-http-request-logger.git</connection>
    <developerConnection>scm:git:ssh://github.com/orionlibs/spring-http-request-logger.git</developerConnection>
</scm>


<licenses>
    <license>
        <name>Apache License, Version 2.0 OR WHATEVER THE LICENSE IS</name>
        <url>http://www.apache.org/licenses/LICENSE-2.0.html</url>
    </license>
</licenses>


<developers>
    <developer>
        <name>My Real Name</name>
        <email>My Real Email Address</email>
    </developer>
</developers>


<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>3.3.0</version>
            <executions>
                <execution>
                    <id>attach-sources</id>
                    <goals>
                        <goal>jar-no-fork</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>


        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-javadoc-plugin</artifactId>
            <version>3.5.0</version>
            <executions>
                <execution>
                    <id>attach-javadocs</id>
                    <goals>
                        <goal>jar</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>


        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-install-plugin</artifactId>
            <version>3.1.1</version>
        </plugin>


        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-deploy-plugin</artifactId>
            <version>3.1.1</version>
        </plugin>


        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>versions-maven-plugin</artifactId>
            <version>2.16.0</version>
            <configuration>
                <generateBackupPoms>false</generateBackupPoms>
            </configuration>
        </plugin>


        <plugin>
            <groupId>org.sonatype.plugins</groupId>
            <artifactId>nexus-staging-maven-plugin</artifactId>
            <version>1.6.13</version>
            <extensions>true</extensions>
            <configuration>
                <serverId>ossrh</serverId>
                <nexusUrl>https://s01.oss.sonatype.org/</nexusUrl>
                <autoReleaseAfterClose>true</autoReleaseAfterClose>
            </configuration>
        </plugin>


        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-gpg-plugin</artifactId>
            <version>3.1.0</version>
            <executions>
                <execution>
                    <id>sign-artifacts</id>
                    <phase>verify</phase>
                    <goals>
                        <goal>sign</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <keyname>0xB5D5824C OR WHATEVER THE HEXADECIMAL CODE OF YOUR GPG KEY IS. SEE STEP 10</keyname>
                <passphraseServerId>ossrh</passphraseServerId>
            </configuration>
        </plugin>


        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-release-plugin</artifactId>
            <version>3.0.1</version>
            <configuration>
                <autoVersionSubmodules>true</autoVersionSubmodules>
                <useReleaseProfile>false</useReleaseProfile>
                <releaseProfiles>release</releaseProfiles>
                <!--<goals>deploy</goals>-->
                <goals>deploy nexus-staging:release</goals>
            </configuration>
        </plugin>
    </plugins>
</build>


<distributionManagement>
    <snapshotRepository>
        <id>ossrh</id>
        <url>https://s01.oss.sonatype.org/content/repositories/snapshots</url>
    </snapshotRepository>
    <repository>
        <id>ossrh</id>
        <url>https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/</url>
    </repository>
</distributionManagement>
```
19. Once your project is completed and you want to publish it to Maven Central, run once again  
"mvn clean deploy" and it will take a minute to publish it. There is also the command  
"mvn clean deploy -P release", but I don't know if it works.
20. You can test immediately if it has been published by importing it to another project you have. If you want to see it in the Maven Central then after about 30 minutes you will see it in  
https://repo1.maven.org/maven2/io/github/my-username/my-repo/  
You can also maange your repos from  
https://s01.oss.sonatype.org/#stagingRepositories  
The login button is on the top-right and the credentials are the same as the JIRA account you created earlier.
21. HAPPY CODING!!!
