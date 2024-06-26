# Ansible Playbook: Rebuild fastx3 Database

This Ansible playbook is designed to rebuild the `fastx3` database by performing the following steps:
1. Stopping the `fastx3` service.
2. Renaming the existing database folder.
3. Copying a new database template.
4. Restarting the `fastx3` service.

## Usage

### Prerequisites
- Ensure you have Ansible installed on your control machine.
```bash
python3 -m pip install --user ansible
```
- Verify that you have the appropriate permissions to manage services and modify files on the target hosts.
- Update your inventory file with the target hosts.

### Playbook Details

#### Playbook Name
`rebuild_fastx3_db.yml`

#### Host Group
`rud`

#### Variables
- `inventory_hostname`: This variable is used to reference the hostname of the target machine.

### Steps

1. **Stop fastx3 Service**
    - Uses the `ansible.builtin.systemd` module to stop the `fastx3` service.
    - On production, replace `fastx3` with the actual service name if different.

2. **Rename Folder**
    - Renames the existing database folder to create a backup.
    - The folder `/d/db/fastx3/{{ inventory_hostname }}` is renamed to `/d/db/fastx3/{{ inventory_hostname }}_backup`.

3. **Copy Database Template**
    - Copies the contents of the database template folder to the new database location.
    - Uses `rsync` to copy files from `/d/db/fastx3/rud_fastx_skel/` to `/d/db/fastx3/{{ inventory_hostname }}/`.
    - On production, replace the source path with the appropriate template folder.

4. **Restart fastx3 Service**
    - Restarts the `fastx3` service using the `ansible.builtin.systemd` module.
    - On production, replace `fastx3` with the actual service name if different.

### Example Playbook

```yaml
---
- name: Rebuild fastx3 db
  hosts: rud
  become: true
  gather_facts: false # true on prod
  any_errors_fatal: true

  tasks:
    - name: Stop fastx3 service
      ansible.builtin.systemd:
        name: fastx3 # remove on prod
        # name: fastx3
        state: stopped
      register: fastx3_status
      failed_when: "'restarted' in fastx3_status.state"
 
    - name: Rename folder
      command: "mv /d/db/fastx3/{{ inventory_hostname }} /d/db/fastx3/{{ inventory_hostname }}_backup"

    - name: Copy db template
      command: "rsync -av /d/db/fastx3/rud_fastx_skel/ /d/db/fastx3/{{ inventory_hostname }}/"
      # command: "rsync -av /d/db/fastx3/ud_fastx_skel/ /d/db/fastx3/{{ inventory_hostname }}/"

    - name: Restart fastx3 service
      ansible.builtin.systemd:
        name: fastx3 # remove on prod
        # name: slurmdb
        state: restarted
      register: fastx3_status
      failed_when: "'stopped' in fastx3_status.state"
```

### Running the Playbook

To run the playbook, use the following command:
```sh
ansible-playbook rebuild_fastx3_db.yml -i your_inventory_file
```

### Notes

- Ensure that the `gather_facts` parameter is set to `true` on production environments if needed.
- Adjust the service name and template paths as per your production environment requirements.

This playbook provides a systematic approach to rebuilding the `fastx3` database, ensuring minimal downtime and data integrity through the use of backups and templating.

---
