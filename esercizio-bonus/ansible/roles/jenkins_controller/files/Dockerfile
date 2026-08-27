# Dockerfile utilizzato per buildare l'immagine custom del controller Jenkins in modo tale da incorporare la configurazione JCasC e i plugin necessari.
# Immagine custom basata su jenkins/jenkins:lts,
FROM docker.io/jenkins/jenkins:lts

# Salta il setup wizard dato che la configurazione è gestita interamente da JCasC.
ENV JAVA_OPTS="-Djenkins.install.runSetupWizard=false"

# Preinstalla i plugin elencati in plugins.txt usando jenkins-plugin-cli,
COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt

# Incorpora la configurazione JCasC nell'immagine.
# La path è sotto /usr/share/jenkins/ref/ (reference directory), NON sotto
# /var/jenkins_home: quest'ultima viene coperta dal named volume a runtime,
# il che maschererebbe il file. La reference directory invece è sicura.
COPY jenkins.yaml /usr/share/jenkins/ref/casc_configs/jenkins.yaml

# Indica a JCasC dove trovare la sua configurazione: il plugin
# configuration-as-code legge questo path all'avvio.
ENV CASC_JENKINS_CONFIG=/usr/share/jenkins/ref/casc_configs/jenkins.yaml