# SHiP Onboarding

This is a compact starting page for a student joining SHiP work at CERN. It is not a replacement for official CERN or SHiP documentation. Use it as a practical order of operations, then follow the linked source pages for the current rules, forms, and service details.

!!! note
    Your exact access depends on your institute, CERN registration category, supervisor, SHiP role, and safety requirements. When this page and an official CERN or SHiP page differ, follow the official page.

## 1. Understand where your work fits

Start by reading the public [SHiP website](https://ship.web.cern.ch/) and the [SHiP software documentation](https://shipsoft.github.io/Documentation/). You do not need to understand every detector or software component on day one.
For software work, also check the [ShipSoft GitHub organization](https://github.com/ShipSoft). Public documentation and public repositories are good entry points, while internal material may require CERN or collaboration access.

A few articles to introduce you to SHiP.
- [The SHiP experiment at the proposed CERN SPS Beam Dump Facility](https://arxiv.org/pdf/2112.01487)

## 2. Sort out CERN registration

Before arrival, confirm your status with your supervisor, team leader, or institute contact. Registration controls many later steps: account creation, site access, insurance questions, safety training, computing eligibility, and protected resources.

Use the [CERN Users Office](https://usersoffice.web.cern.ch/) as the official starting point. In particular, check the [pre-arrival formalities](https://usersoffice.web.cern.ch/pre-arrival-formalities-and-information/) and [first-day onboarding](https://usersoffice.web.cern.ch/first-day-cern-and-onboarding) pages.

Ask your supervisor:

- Which documents your institute must provide.
- Whether you need a CERN access card.
- Whether you need safety courses, dosimetry, lab access, underground access, or radiation-area access.
- Who approves SHiP mailing-list, group, repository, and meeting access.

## 3. Activate the basic services

Once your CERN account is available, verify the services you will use regularly:

| Service | What to check |
| --- | --- |
| CERN Single Sign-On | You can sign in to CERN web services. |
| Users Portal | Your profile, affiliation, and service information look correct. |
| Groups and mailing lists | You have requested or received the memberships your task needs. |
| CERNBox | You can store and share ordinary documents. |
| GitLab or GitHub | You can access the repository used by your task. |
| Indico | You can open the meetings, agendas, and materials for your group. |
| Mattermost or email | You know where day-to-day discussion happens. |
| LXPLUS, EOS, CVMFS, batch | You have access to CERN computing resources. |

Official service pages:

- [CERN Authentication and Authorization docs](https://auth.docs.cern.ch/)
- [CERNBox docs](https://cernbox.docs.cern.ch/)
- [Indico docs](https://indico.docs.cern.ch/)
- [Mattermost at CERN docs](https://mattermost.docs.cern.ch/)
- [CERN Batch docs](https://batchdocs.web.cern.ch/)
- [CERN Service Portal](https://cern.service-now.com/service-portal)

## 4. Prepare CERN computing and training tools

Most SHiP software work eventually touches CERN computing services. You do not need to master them all in the first week, but you should know what each one is for and how to verify access.

### AFS access at CERN

AFS is CERN's authenticated shared filesystem for Unix-style home directories and some collaboration areas. On CERN-managed Linux machines and on LXPLUS, AFS is normally mounted for you. You can check whether you are already subscribed to AFS in the CERN Computing Resources Portal: https://resources-portal.web.cern.ch/.

If you are subscribed, the usual first check will be:
```bash
ssh <cern-username>@lxplus.cern.ch
ls /afs/cern.ch/user/${USER:0:1}/${USER}
```

Ask your supervisor or the [CERN Service Portal](https://cern.service-now.com/service-portal) when:

- Your CERN account works in the browser but not on LXPLUS.
- You can read AFS but cannot write to the expected directory.
- You are unsure whether a task should use AFS, EOS, CERNBox, CVMFS, or Git.

### CERN SWAN

[SWAN](https://swan.cern.ch/) is CERN's web-based Jupyter notebook service. It lets you run notebooks in a browser using CERN authentication, with access to common CERN storage and software services. Start from the [SWAN documentation](https://swan.docs.cern.ch/) if you need details on sessions, storage, software environments, sharing, or batch/Spark integrations.

SWAN is useful for:

- Trying small Python, ROOT, or analysis examples without configuring a full local environment.
- Opening training notebooks directly from CERN-supported courses.
- Making quick validation plots from files you can reach from CERN storage.
- Sharing a short, reproducible notebook with a supervisor.

Do not use SWAN as the only record of important work. Keep analysis code, configuration, and notes in the repository or documentation location agreed with your supervisor.

### Training material

For general HEP software skills, use the [HSF Training Center](https://hsf-training.org/training-center/). It collects training material for the high-energy physics community, including shell, Git, Python, C++, containers, CI, ROOT, Scikit-HEP, and analysis-preservation topics.

For ROOT, start from the official [ROOT@CERN Learn page](https://root.cern/learn/), which links to the current ROOT courses, exercises, tutorials, manual, API documentation, and ROOT forum. The ROOT courses can be run through SWAN if you have a CERN computing account.

## 5. Set up your first technical task

Do not begin by trying to configure every possible CERN and SHiP tool. Start from one reproducible task agreed with your supervisor.

Write down:

- The goal: for example, reproduce a simulation step, update documentation, test geometry configuration, make a validation plot, or prepare a short note.
- The repository or documentation page involved.
- The expected output: merge request, plot, log file, presentation, dataset, or written summary.
- The environment: laptop, LXPLUS, container, CVMFS release, or CI.
- The review path: supervisor, working group, repository maintainer, or SHiP contact.

For a first contribution, prefer something small and observable. Good first tasks include reproducing a documented command sequence, fixing outdated setup notes, adding a missing example, recording software versions, or improving a troubleshooting section.

## 6. Use this first-week checklist

- Confirm your CERN registration category and institute paperwork.
- Confirm your SHiP contact person and first task.
- Activate CERN account access and sign in successfully.
- Request required groups, mailing lists, meetings, and repository access.
- Identify the relevant SHiP meeting on Indico.
- Find the canonical repository or documentation page for your work.
- Decide whether you should use a laptop, LXPLUS, container, CVMFS, or another environment.
- Check whether your task needs AFS, EOS, CERNBox, CVMFS, SWAN, or batch access.
- Reproduce one small documented command or workflow.
- Record the commands, software versions, input files, and output location.

## 7. Know where to ask

Use the narrowest support path that fits the problem:

| Problem | First place to ask |
| --- | --- |
| Project scope, physics goal, review expectations | Your supervisor or SHiP working group |
| CERN registration, contracts, access cards, local life | [CERN Users Office](https://usersoffice.web.cern.ch/) |
| CERN service incident or access problem | [CERN Service Portal](https://cern.service-now.com/service-portal) |
| Repository permissions or merge-request review | Repository maintainers or your SHiP contact |
| Software setup, simulation, geometry, packaging | [SHiP software documentation](https://shipsoft.github.io/Documentation/), [SHiP software Mattermost Channel](https://mattermost.web.cern.ch/ship/channels/software)|
