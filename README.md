
[![Maintainability](https://api.codeclimate.com/v1/badges/700169481b27774825a2/maintainability)](https://codeclimate.com/github/Milfist/Drone/maintainability) [![Test Coverage](https://api.codeclimate.com/v1/badges/700169481b27774825a2/test_coverage)](https://codeclimate.com/github/Milfist/Drone/test_coverage)
# Configure-TravisCI-CodeClimate-Maven-Java

## Description
 Explanation of how to configure a Java project with Travis CI, CodeClimate Coverage and Maven

### What is the problem?

The CodeClimate coverage badge, with this particular configuration, is quite complicated to achieve. There is very little documentation, just a few unclear posts and the documentation of CodeClimate is ... bad, little updated and is pure disinformation. While other code evaluation platforms such as Sonarcloud or Codecov, are very simple and with good documentation, with CodeClimate we have to roll up and get to the bottom.

### Let's do it

#### pom.xml

The first thing we need is a plugin to test the coverage of our project. It is compatible with [Jacoco][0] and [Cobertura][1], I chose the first option to connect it and because I had it implemented for Codecov.

            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.8.1</version>
                <executions>
                    <execution>
                        <id>prepare-agent</id>
                        <goals>
                            <goal>prepare-agent</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>report</id>
                        <phase>prepare-package</phase>
                        <goals>
                            <goal>report</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>post-unit-test</id>
                        <phase>test</phase>
                        <goals>
                            <goal>report</goal>
                        </goals>
                        <configuration>
                            <!-- Sets the path to the file which contains the execution data. -->
                            <dataFile>target/jacoco.exec</dataFile>
                            <!-- Sets the output directory for the code coverage report. -->
                            <outputDirectory>target/jacoco-ut</outputDirectory>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            
The problems begin...

I'm going to introduce another plugin to be able to launch Ant tasks, why? CodeClimate needs the coverage format that is generated in jacoco.xml, but the routes that are inside it, cause problems and you have to modify the routes of the sources that describe in the xml.          

##### Trace:

            0.24s$ ./cc-test-reporter format-coverage -d -t jacoco ./target/jacoco-ut/jacoco.xml
            time="2018-04-18T14:45:13Z" level=debug msg="coverage path ./target/jacoco-ut/jacoco.xml" 
            time="2018-04-18T14:45:13Z" level=debug msg="using formatter jacoco" 
            time="2018-04-18T14:45:13Z" level=debug msg="checking search path ./target/jacoco-ut/jacoco.xml for jacoco formatter" 
            time="2018-04-18T14:45:13Z" level=debug msg="couldn't load committed at from ENV, trying git..." 
            time="2018-04-18T14:45:13Z" level=info msg="trimming with prefix /home/travis/build/Milfist/Drone/" 
            time="2018-04-18T14:45:13Z" level=debug msg="getting fallback blob_id for source file com/drone/driver/Driver.java" 
            time="2018-04-18T14:45:13Z" level=error msg="failed to read file com/drone/driver/Driver.java\nopen com/drone/driver/Driver.java: no such file or directory" 
            Error: open com/drone/driver/Driver.java: no such file or directory
            Usage:
              cc-test-reporter format-coverage [coverage file] [flags]

##### Generated:
            
            <class name="com/drone/driver/Driver">

##### Necessary:

            <class name="src/main/java/com/drone/driver/Driver">

##### Plugin and task

            <plugin>
                <artifactId>maven-antrun-plugin</artifactId>
                <executions>
                    <execution>
                        <phase>test</phase>
                        <configuration>
                            <tasks>
                                <replace file="target/jacoco-ut/jacoco.xml" token="com/" value="src/main/java/com/"/>
                            </tasks>
                        </configuration>
                        <goals>
                            <goal>run</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

###### Note: If you want to verify this for yourselves, forget this last and resort to this when you need it.

#### .travis.yml
This configuration is different from what you will find in the official documentation. Let's see it:

            language: java
            dist: trusty
            jdk: oraclejdk9
            env:
              global:
                - CC_TEST_REPORTER_ID=[YOUR CODE COVERAGE]
            before_script:
              - curl -L https://codeclimate.com/downloads/test-reporter/test-reporter-latest-linux-amd64 > ./cc-test-reporter
              - chmod +x ./cc-test-reporter
              - ./cc-test-reporter before-build
            script:
              - mvn test
            after_script:
              - ./cc-test-reporter format-coverage -d -t jacoco ./target/jacoco-ut/jacoco.xml
              - ./cc-test-reporter upload-coverage -d

#####Part by part

The first three lines are common and you will already know them.
The env block is the coverage token that relates the execution to CodeClimate. This token is only for this, not being used to post against your repository or anything weird. I have read this information in the CodeClimate forums written by them. I have tried to do it through this that I show you:

            addons:
              code_climate:
                repo_token: 
                  secure: "hkpU5hbhM7Xhn8OG..."


Installing travis in your console, you can code your tokens, but in this case, it has not worked for me. You need to have the variable CC_TEST_REPORTER_ID as input.
###### I'm still investigating this point, but to prove it, we're worth it for the moment.

##### before_script:
This section is the same as what is shown in the official documentation, nothing to comment.

##### after_script:
In this section there are changes. We use the downloaded tool to format with the jacoco.xml, which will generate a codeclimate.json and that we will upload with the second line.
Keep in mind the routes in which you configure the report.


I hope it helps you. I will continue investigating the weak points of this configuration to make it more robust.


### Configured and operational project

Click [HERE][2]
 
# By:
 
[![alt text](https://github.com/Milfist/Docs/blob/master/milfist.JPG)][1]
 
[0]: http://www.jacoco.org/jacoco/trunk/doc/
[1]: http://cobertura.github.io/cobertura/
[2]: https://github.com/Milfist/Drone

