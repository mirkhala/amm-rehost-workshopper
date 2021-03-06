# Deploy the monolith to OpenShift

We now have a fully migrated application that we tested locally. Let's deploy it to OpenShift.

**1.** Add a OpenShift profile

Open the `pom.xml` file.

At the `<!-- TODO: Add OpenShift profile here -->` we are going to add a the following configuration to the pom.xml

~~~xml
<profile>
  <id>openshift</id>
  <build>
      <plugins>
          <plugin>
              <artifactId>maven-war-plugin</artifactId>
              <version>2.6</version>
              <configuration>
                  <webResources>
                      <resource>
                          <directory>${basedir}/src/main/webapp/WEB-INF</directory>
                          <filtering>true</filtering>
                          <targetPath>WEB-INF</targetPath>
                      </resource>
                  </webResources>
                  <outputDirectory>deployments</outputDirectory>
                  <warName>ROOT</warName>
              </configuration>
          </plugin>
      </plugins>
  </build>
</profile>
~~~

**2.** Create the OpenShift project

First, open the OpenShift Console in the browser: https://microservices.msa#.rhcol.com

![OpenShift Console]({% image_path /scenario1/image28.png %}){:width="550 px"}

Login using:

* Username: admin
* Password: admin

You will see the OpenShift landing page:

![OpenShift Console]({% image_path /scenario1/image62.png %}){:width="650 px"}

Click Create Project, fill in the fields, and click Create:

* Name: `coolstore-dev`
* Display Name: `Coolstore Monolith - Dev`
* Description: _leave this field empty_

NOTE: YOU MUST USE `coolstore-dev` AS THE PROJECT NAME, as this name is referenced later on and you will experience failures if you do not name it `coolstore-dev`!

![OpenShift Console]({% image_path /scenario1/image25.png %}){:width="391 px"}

Click on the name of the newly-created project:

![OpenShift Console]({% image_path /scenario1/image6.png %})

This will take you to the project overview. There's nothing there yet, but that's about to change.

**3.** Deploy the monolith

We'll use the CLI to deploy the components for our monolith. To deploy the monolith template using the CLI, execute the following commands:

Firstly, you will need to open the terminal window and make a login into Openshift Container Platform. The web interface provides a ready to use oc command with the token needed.

![]({% image_path /scenario1/image30.png %}){:width="250 px"}

Just paste it in the terminal window:

~~~shell
oc login https://microservices.msa#.rhcol.com
~~~

Switch to the terminal window in project you created earlier:

~~~shell
oc project coolstore-dev
~~~

And finally deploy template:

~~~shell
oc process -f https://raw.githubusercontent.com/coolstore/monolith/master/src/main/openshift/template.json | oc create -f -
oc new-app coolstore-monolith-binary-build
~~~

This will deploy both a PostgreSQL database and JBoss EAP, but it will not start a build for our application.

Then open up the Monolith Overview page at

http://www-coolstore-dev.apps.msa#.rhcol.com and verify the monolith template items are created:

![OpenShift Console]({% image_path /scenario1/image16.png %}){:width="650 px"}

You can see the components being deployed on the Project Overview, but notice the No deployments for Coolstore. You have not yet deployed the container image built in previous steps, but you'll do that next.

**4.** Deploy application using Binary build

In this development project we have selected to use a process called binary builds, which means that we are going to build locally and just upload the artifact \(e.g. the .war file\). The binary deployment will speed up the build process significantly.

First, build the project once more using the openshift Maven profile, which will create a suitable binary for use with OpenShift \(this is not a container image yet, but just the .war file\). We will do this with the oc command line.

Build the project:

Using JBoss Developer studio, right click in the pom.xml, navigate to **Run As → Maven build…**  
  


![]({% image_path /scenario1/image59.png %}){:width="650 px"}

Add clean package -Popenshift as the goals to this maven build

![]({% image_path /scenario1/image57.png %}){:width="650 px"}

You can also run the command below in the terminal window if you prefer

~~~shell
mvn clean package -Popenshift
~~~

Wait for the build to finish and the `BUILD SUCCESS` message!

Go to Openshift Explorer tab and add a new connection using its wizard:

![]({% image_path /scenario1/image46.png %}){:width="650 px"}

Enter the master node url provided by the instructor https://microservices.msa#.rhcol.com, select Basic protocol and enter with username `admin` and password `admin`.

![]({% image_path /scenario1/image10.png %}){:width="450 px"}

If asked, accept the SSL certificate.

![]({% image_path /scenario1/image7.png %}){:width="450 px"}

Then click finish.

You will see the Openshift connection in Openshift Explorer tab.

![]({% image_path /scenario1/image54.png %}){:width="524 px"}

And finally, start the build process that will take the `.war` file and combine it with JBoss EAP and produce a Linux container image which will be automatically deployed into the project, thanks to the _DeploymentConfig_ object created from the template.

In terminal window, certify you are in the coolstore-dev project

~~~shell
oc get project
~~~

Start the coolstore application build

~~~shell
oc start-build coolstore --from-file=${HOME}/projects/monolith/deployments/ROOT.war
~~~

![]({% image_path /scenario1/image34.png %}){:width="650 px"}

Check the OpenShift web console and you'll see the application being built:

![OpenShift Console]({% image_path /scenario1/image39.png %}){:width="650 px"}

You will also see the Openshift Explorer tab in JBoss Developer Studio updated.

![]({% image_path /scenario1/image37.png %}){:width="550 px"}

In terminal window, wait for the build and deploy to complete:

~~~shell
oc rollout status -w dc/coolstore
~~~

This command will be used often to wait for deployments to complete. Be sure it returns success when you use it! You should eventually see replication controller "coolstore-1" successfully rolled out.

If the above command reports `Error from server`\(ServerTimeout\) then simply re-run the command until it reports success!

When it's done you should see the application deployed successfully with blue circles for the database and the monolith:

![OpenShift Console]({% image_path /scenario1/image38.png %}){:width="650 px"}

Test the application by clicking on the Route link at

http://www-coolstore-dev.apps.msa#.rhcol.com , which will open the same monolith Coolstore in your browser, this time running on OpenShift:

![OpenShift Console]({% image_path /scenario1/image53.png %}){:width="650 px"}



