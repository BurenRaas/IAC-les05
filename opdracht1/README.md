# Les 05 - Opdracht 1

## Opdracht

In deze opdracht wordt een Ansible playbook geschreven dat:
- Het pakket `apache2` installeert **zonder gebruik van de `apt` module**.
- Een `changed` melding genereert als het installeren gelukt is.
- Een aparte taak bevat die **expres faalt**, zodat Ansible een foutmelding weergeeft.


## Commando's
terraform apply -auto-approve # Omgeving opbouwen
ansible-playbook -i inventory.yml playbook.yml # Ansible ingang zetten


## Apache installeren zonder apt module:
In roles/webserver/tasks/main.yml
- name: Installeer apache2 zonder apt module
  command: apt-get install -y apache2
  changed_when: true # force changed notificatie

Apache wordt geïnstalleerd via command met een "changed" melding. Ik kon ook DNF of YUM als package manager gebruiken, maar dit kreeg ik niet aan de praat. Dus dan maar de command module gebruikt (die dan wel apt-get aanroept).

## Forceer fout
In roles/databaseserver/taks/main.yml
- name: Start python3-pymysql
  service:
    name: python3-pymysql
    state: started
    enabled: yes

Ansible toont een foutmelding bij de gefaalde taak op de databaseserver:
fatal: [192.168.20.48]: FAILED! => {"changed": false, "msg": "Could not find the requested service python3-pymysql: host"}

## Bronnen
ansible.builtin.command module https://docs.ansible.com/ansible/latest/collections/ansible/builtin/command_module.html

Code gebruikt uit les 4.

AI prompt: genereer een readme.md voor deze code:
https://chatgpt.com/share/6831d7e5-24dc-8007-afb2-9af4d7bac823

Ruben Baas s1190828