# Les 05 - Opdracht 2

## Opdracht
- Installeer een lokale GitHub Runner en voeg deze toe aan je repository.
- Maak een GitHub workflow aan in .github/workflows/, die automatisch wordt uitgevoerd bij een push.
- Schrijf een Ansible playbook dat een package zoals apache2 of nginx installeert.
- Richt meerdere CI/CD stages in, waaronder een codecontrole en de uitvoering van het playbook.
- Laat het playbook automatisch draaien via de lokale runner zodra er een wijziging wordt gepusht.

## terraform aanpassingen

De onderstaande Terraform-code is toegevoegd aan main.tf om automatisch de IPs aan inventory.ini en  known_hosts toe te voegen na het aanmaken van de VMs:

--
resource "null_resource" "generate_inventory_and_known_hosts" {
  provisioner "local-exec" {
    command = <<EOT
echo "[webserver]" > inventory.ini
%{ for ip in esxi_guest.webserver[*].ip_address ~}
echo "${ip} ansible_user=student ansible_ssh_private_key_file=~/.ssh/iac" >> inventory.ini
ssh-keyscan -H ${ip} >> ~/.ssh/known_hosts
%{ endfor ~}

echo "" >> inventory.ini
echo "[databaseserver]" >> inventory.ini
echo "${esxi_guest.databaseserver.ip_address} ansible_user=student ansible_ssh_private_key_file=~/.ssh/iac" >> inventory.ini
ssh-keyscan -H ${esxi_guest.databaseserver.ip_address} >> ~/.ssh/known_hosts
EOT
  }

  depends_on = [
    esxi_guest.webserver,
    esxi_guest.databaseserver
  ]
}

--

## Ansible-playbook
Het Ansible Playbook uit opdracht 1 van les 5 wordt gebruikt. In het kort:
  Het playbook (playbook.yml) installeert de volgende pakketten:
  apache2 op alle hosts in de groep [webserver]
  mysql-server op hosts in [databaseserver]

### CI/CD Pipeline (ci.yml) uitleg

De ci.yml workflowbestand is geplaatst in .github/workflows/ci.yml en voert de volgende stappen uit:

- Trigger: de workflow start automatisch bij een push naar de main branch.
- Checkout: haalt de nieuwste versie van de repository op.
- Terraform Init & Apply: initialiseert Terraform en bouwt de vms op.
- Ansible Playbook: voert het ansible-playbook uit dat apache2 (op de webserver) en mysql (op de databaseserver) installeert.

Er is een lokale GitHub Runner gebruikt zodat de workflow draait op een eigen systeem (ontwikkel-VM in Skylab), in plaats van op GitHub’s cloudinfrastructuur.


## Bronnen
Code gebruikt uit opdracht 1 van deze les.


Adding a self-hosted runner to a repository:
https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners#adding-a-self-hosted-runner-to-a-repository
Github action syntax:
https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions

AI prompts:
Met terraform apply ook automatisch de IPs toevoegen aan known_hosts
https://chatgpt.com/share/6845a27c-0e3c-8007-97dc-f27b0130b182
Maak deze readme.md af
https://chatgpt.com/share/6845a90b-7020-8007-bb02-6c35929933e7

Ruben Baas s1190828