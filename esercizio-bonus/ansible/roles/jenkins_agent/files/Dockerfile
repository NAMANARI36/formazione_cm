# Dockerfile utilizzato per buildare l'immagine custom dell'Agent Jenkins, con l'obiettivo di renderlo capace di eseguire 
# build attraverso la podman CLI usata come client remoto verso il socket Podman dell'host.
# La build delle immagini avviene sull'host(via socket).
# Immagine custom basata su jenkins/inbound-agent:latest
FROM docker.io/jenkins/inbound-agent:latest

# Passa a root per installare pacchetti
USER root

# Installa podman
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        podman \
        ansible \
        openssh-client 

# Torna all'utente non privilegiato per l'esecuzione dell'agent.
USER jenkins