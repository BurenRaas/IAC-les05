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

### CI/CD Pipeline uitleg

De workflow is gedefinieerd in `.github/workflows/ci.yml` en bestaat uit drie opeenvolgende jobs: `lint`, `terraform`, en `deploy`. De workflow wordt automatisch uitgevoerd bij een `push` of `pull request` naar de `main` branch.

#### 🔄 Workflow-triggers
```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
```

#### Job 1 – Lint: Ansible codecontrole

Deze stap valideert de structuur van het Ansible-playbook met `ansible-lint`. Fouten in de lintcheck worden **gelogd maar onderbreken de workflow niet**, dankzij `continue-on-error: true`.

Acties in deze job:
- Checkt de laatste code uit met `actions/checkout@v3`
- Installeert Python en Ansible via `actions/setup-python@v5`
- Installeert `ansible-lint`
- Voert `ansible-lint playbook.yml` uit in de map `./opdracht2`. Ansible lint controleert de syntanx en best practices van de code.

#### Job 2 – Terraform: Provisioning

Deze job draait **pas na de lint-check** (`needs: lint`). Hierin wordt de infrastructuur aangemaakt met Terraform. 

Acties in deze job:
- Initialiseert Terraform (`terraform init`)
- Voert de configuratie uit (`terraform apply --auto-approve`)
- Terraform genereert automatisch `inventory.ini` en voegt IP-adressen toe aan `~/.ssh/known_hosts` via een `null_resource` met `local-exec`

De configuratie draait binnen de `./opdracht2` directory. Want in deze map staan de bestanden voor deze opdracht.

#### Job 3 – Deploy: Ansible-configuratie

De derde job draait **na succesvolle provisioning** (`needs: terraform`). In deze stap wordt het Ansible-playbook uitgevoerd dat software installeert op de nieuw aangemaakte servers.

Acties in deze job:
- Checkt opnieuw de code uit
- Voert `ansible-playbook -i inventory.ini playbook.yml` uit

Het playbook:
- Installeert `apache2` op hosts in de groep `[webserver]`
- Installeert `mysql-server` op hosts in `[databaseserver]`

Deze job draait op een self-hosted GitHub Runner die is geïnstalleerd op een ontwikkel-VM binnen Skylab, zodat lokale communicatie met de virtuele machines mogelijk is.

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