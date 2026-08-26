# formazione_cm

La chiave pubblica per il ruolo che crea i container per gli OS deve essere passata al lancio del playbook.
Esempio:

ansible-playbook -i inventory setup-step3-playbook.yml \
  -e "container_images_ssh_pubkey='$(cat ~/.ssh/id_ed25519.pub)'"