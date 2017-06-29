Use case: Jenkins contiuous integration server
==============================================

Jenkins has an Ubuntu repository which will provide the latest vesion to
date. Install Jenkins by running the following commands:

::
    wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
    echo "deb http://pkg.jenkins-ci.org/debian binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list
    sudo apt-get update
    sudo apt-get install jenkins

