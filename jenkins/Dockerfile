# Jenkins configured as code: plugins, admin user, kubeconfig credential and pipeline job
# are provisioned at startup, no setup wizard.
FROM jenkins/jenkins:lts-jdk21
ENV JAVA_OPTS=-Djenkins.install.runSetupWizard=false \
    CASC_JENKINS_CONFIG=/casc.yaml
RUN jenkins-plugin-cli --plugins \
      workflow-aggregator git credentials-binding \
      configuration-as-code job-dsl pipeline-graph-view
COPY casc.yaml /casc.yaml
