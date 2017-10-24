
Custom Insights Rules as Ansible Playbooks, Ansible Playbooks that act as Insights Rules

This is just the beginnings of a proof of concept.

The general idea is that we run mostly normal Ansible playbooks in --check mode, and
interpret the result of each task in the playbook as a conformance test, and then forward
those interpreted results to Insights for display.

Until we have a way to forward the result to Insights, we just print the interpreted results
out.

There are currently just two example Insights Rules written as Ansible Playbooks:
   insights-check-mode-playbooks/fips-mode-check.yml
     Which checks that a system is in FIPS mode.
   
   insights-check-mode-playbooks/prelink-absent-check.yml
     Which checks that a system does not have the prelink package installed.

Use these playbooks with:
  ansible-playbook --check --limit=<HOST PATTERN> <CHECK PLAYBOOK>

where
  <HOST PATTERN> is a comma separated list of hosts to run the check against
  <CHECK PLAYBOOK> is one of 
    insights-check-mode-playbooks/fips-mode-check.yml
    insights-check-mode-playbooks/prelink-absent-check.yml

You can use your development machine as <HOST PATTERN>, but for fips mode, the results will
probably be booring.

You can create new <CHECK PLAYBOOKS> but, for the time being, all playbooks must be in the
insights-check-mode-playbooks directory to take advantage of "Insights Check Mode".


Insights Check Mode

When a playbook is run in "Insights Check Mode", a new "CHECKMODE SUMMARY" is added to the
end of the normal output from running the playbook.

In the example below are two RHEL 6.6 machines, one with FIPS mode enabled one without.

$ ansible-playbook --check --limit=gavin-rhel66-nofips,gavin-rhel66-yesfips insights-check-mode-playbooks/fips-mode-check.yml 

PLAY [all] **********************************************************************************************

TASK [Gathering Facts] **********************************************************************************
ok: [gavin-rhel66-nofips]
ok: [gavin-rhel66-yesfips]

TASK [fips mode must be enabled] ************************************************************************
changed: [gavin-rhel66-nofips]
ok: [gavin-rhel66-yesfips]

PLAY RECAP **********************************************************************************************
gavin-rhel66-nofips        : ok=2    changed=1    unreachable=0    failed=0   
gavin-rhel66-yesfips       : ok=2    changed=0    unreachable=0    failed=0   


CHECKMODE SUMMARY ***************************************************************************************
gavin-rhel66-nofips
    no : fips mode must be enabled
gavin-rhel66-yesfips
    ok : fips mode must be enabled


In the CHECKMODE SUMMARY, the final "no" and "ok" are the important bits.  The label "no" is a
big red "X", no this system is NOT in fips mode.  The label "yes" is a green checkmark, yes the
system is in fips mode.
