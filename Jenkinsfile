//node('main') {
pipeline {
	agent any
	stages {
	stage('Poll') {
		steps{
		checkout scm
		}
	}
	stage('Build and Unit Test'){
		steps{
		sh 'mvn clean verify -DskipITs=true';
		junit '**/target/surefire-reports/TEST-*.xml'
		archive 'target/*.jar'
		}
	}
	stage('SonarQube Scan') {
		steps{
		// Create JaCoCo code coverage report
		sh 'mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent install -Dmaven.test.failure.ignore=false'
		withSonarQubeEnv('Default SonarQube server') {
			sh 'mvn clean verify sonar:sonar -Dsonar.projectName=example-project -Dsonar.projectKey=example-project -Dsonar.projectVersion=$BUILD_NUMBER';
		}
		}
	}
	stage('SonarQube Quality Gate') {
		steps{
		timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
			step{
			def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
				step{
				if (qg.status != 'OK') {
				error "Pipeline aborted due to quality gate failure: ${qg.status}"
				}}
			}
		}
		}
	}
	stage ('Integration Test'){
		steps{
		sh 'mvn clean verify -Dsurefire.skip=true';
		junit '**/target/failsafe-reports/TEST-*.xml'
		archive 'target/*.jar'
		}
	}
	}
	
}
