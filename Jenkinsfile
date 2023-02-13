pipeline {
    agent { node { label 'SILVER' } }
    tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
	}

    environment {
        registryCredential = 'ecr:us-east-1:awscreds'
        appRegistry = "750232146652.dkr.ecr.us-east-1.amazonaws.com/vprofileappimg"
        vprofileRegistry = "https://750232146652.dkr.ecr.us-east-1.amazonaws.com"
        cluster = "vprofile"
        service = "vprofileappsvs"
    
