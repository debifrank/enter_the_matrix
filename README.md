# BSides Augusta 2021

[![ETM - BSidesAugusta2021](https://img.youtube.com/vi/kDKyXeqeX9c/0.jpg)](https://www.youtube.com/watch?v=kDKyXeqeX9c)

# Table of Contents

   * [Deploying ETM](#deploying-etm)
      * [Installing Dependencies](#installing-dependencies)
         * [Update](#update)
         * [Install Vim](#install-vim)
         * [Install GIT](#install-git)
         * [Install DOS2Unix](#install-dos2unix)
         * [Instal Docker Engine](#install-docker-engine)
      * [Setting up your environment](#setting-up-your-environment)
         * [Creating directories](#creating-directories)
         * [Clone ETM](#clone-etm)
         * [Edit docker-compose.yaml](#edit-docker-composeyaml)
         * [Edit AppSettings.JSON](#edit-appsettingsjson)
         * [SSL Certificate](#ssl-certificate)
         * [Nginx Config](#nginx-config)
      * [Build](#build)
         * [.NET SDK](#net-sdk)
         * [Build the Project](#build-the-project)
      * [Deploy](#deploy)
   * [ETM Usage](#etm-usage)
      * [Creating Users](#creating-users)
      * [Editing Users](#editing-users)
      * [Deleting Users](#deleting-users)
      * [Creating an Assessment](#creating-an-assessment)
      * [Creating a Scenario](#creating-a-scenario)
      * [Creating Events](#creating-events)
      * [Editing an Event](#editing-an-event)
         * [NIST Description Helper](#nist-description-helper)
         * [MITRE ATT&amp;CK IDs](#mitre-attck-ids)
         * [Threat Sources](#threat-sources)
         * [Relevance to Findings](#relevance-to-findings)
         * [Automatic Risk Calculation](#automatic-risk-calculation)
            * [How is Risk Calculated](#how-is-risk-calculated)
         * [Finding Reference/Vulnerability Severity and Predisposing Condition/Pervasiveness of Predisposing Condition](#finding-referencevulnerability-severity-and-predisposing-conditionpervasiveness-of-predisposing-condition)
         * [Graph Elements](#graph-elements)
            * [Entity](#entity)
            * [Description](#description)
            * [Preceded By](#preceded-by)
      * [Creating a Template](#creating-a-template)
      * [Editing a Template](#editing-a-template)
      * [Exporting Graphs](#exporting-graphs)
      * [Exporting Threat Matrix Spreadsheet](#exporting-threat-matrix-spreadsheet)
      * [Exporting Threat Matrix PDF](#exporting-threat-matrix-pdf)
      * [Creating Threat Trees](#creating-threat-trees)
   * [Methodologies](#methodologies)
      * [Simple Threat Source](#simple-threat-source)
      * [Dynamic Threat Source](#dynamic-threat-source)
      * [<Insert Threat Source Methodology> and Sophisticated Intel](#insert-threat-source-methodology-and-sophisticated-intel)

# Deploying ETM

This installation guide has been tested on fresh installs of Ubuntu 18.04 and 20.04.

## Installing Dependencies

ETM is written with .NET Core, and is designed to run on Linux systems.
ETM also is designed to work within Docker containers.

### Update

* `sudo apt update`
* `sudo apt upgrade`

### Install Vim

* `sudo apt install vim`

### Install GIT

* `sudo apt install git`

### Install DOS2Unix

* `sudo apt install dos2unix`

### Install Docker Engine

* `sudo apt update`
* `sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common`
* `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`
* `sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"`
* `sudo apt update`
* `sudo apt install docker-ce docker-ce-cli containerd.io docker-compose`

## Setting up your environment

### Creating directories 

* /var/matrix
* /var/matrix/app
* /var/matrix/mongo
* /var/matrix/mongo/db
* /var/matrix/mongo/configdb

### Clone ETM

* `cd /var/matrix/app`
* `git clone https://github.com/blacklanternsecurity/enter_the_matrix.git`

### Edit docker-compose.yaml

* Change the following line to a unique password (alphanumeric)
* `- MONGO_INITDB_ROOT_PASSWORD=CHANGEMESUCKAH`

### Edit AppSettings.JSON

* `cd /var/matrix/app/enter_the_matrix`
* `vim appsettings.json`
* Alter the ConnectionString to use your password for the MongoDB container
* Replace the "Ldap" fields with your LDAP configuration
* Replace the "LocalAuthSettings" with your desired administrative account credentials

### Edit LdapAuthenticationService.cs

* `cd /var/matrix/app/enter_the_matrix/Services`
* `vim LdapAuthenticationService.cs`
* Alter line `61` and replace `OU=YOURORG,DC=REPLACEME,DC=ANDME` with the appropriate information for your organization

### SSL Certificate

* Place your SSL certificate at `/var/matrix/app/enter_the_matrix/matrix.cer`
* Place your SSL key at `/var/matrix/app/enter_the_matrix/matrix.key`
* To create your own self-signed certificate and key:
  + `sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout matrix.key -out matrix.cer`

### Nginx Config

* Alter `enter_the_matrix.conf` and replace `YOURDOMAIN` with your domain if you are using one

For whatever reason the nginx configuration does not play nicely coming from a Windows development environment even when specifically telling GIT to convert to LF end-of-line format. So, do the following:

* `dos2unix /var/matrix/app/enter_the_matrix/enter_the_matrix.conf`

## Build

### .NET SDK

If you are using Ubuntu 18.04:

* `wget https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb`

Ubuntu 20.04:

* `wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb`

* `sudo dpkg -i packages-microsoft-prod.deb`
* `sudo apt-get install -y apt-transport-https`
* `sudo apt-get update`
* `sudo apt-get install -y dotnet-sdk-5.0`

### Build the project

* `sudo cd /var/matrix/app/enter_the_matrix`
* `sudo dotnet publish --configuration Release`

## Deploy

* `cd /var/matrix/app/enter_the_matrix`
* `docker-compose up -d`

# ETM Usage

![Login Portal](readme-images/Login.png)

Access to all parts of the application requires authentication first.

## Creating Users

![User Management](readme-images/UserManagement.png)

If you do not have LDAP authentication in place already, an administrative user can create user accounts within the application. From the login screen, select the "ADMIN" authentication option and sign in with the credentials set in the appsettings.json file during deployment.

Once you are authenticated as an administrator, you will be brought to the User Management page. Selecting "CREATE" will prompt you to input a new Username, Display Name, Given Name and Password.

Display Names are used in the application as part of the CSS styling. Given Names are used when creating Assessments/Scenarios/Events to tag activities to users.

You can also get to this page by clicking the "Admin" link at the top of every page after a person is authenticated. If the user is not in the "Admin" role, then they will be loggeed out and brought to the login screen again in order to authenticate as an "Admin" user.

## Editing Users

Selecting the "Edit" link next to each user will bring up a prompt to edit the Display Name, Given Name and Password for a user. The Username is not able to be changed.

## Deleting Users

Selecting the "Delete" link next to each user will bring up a confirmation prompt before proceeding to remove the user from the database.

## Creating an Assessment

![Assessments_Page](readme-images/Assessments.png)

Once you are logged in properly, navigate to "Assessments" and then click the "CREATE ASSESSMENT" button. Supply a title for the assessment you are creating and click "CREATE".

## Creating a Scenario

![Scenarios Page](readme-images/Scenarios.png)

After creating an assessment you will be brought to the Scenario page. You can also get to the Scenario page later by selecting the appropriate Assessment.

To create a new Scenario, click "CREATE SCENARIO" and provide a title for the new Scenario.

From the Scenario page, you are also able to edit the title of the Assessment by clicking "EDIT ASSESSMENT" or delete the Assessment by clicking "DELETE ASSESSMENT".Clicking the "THREAT TREES" button will take allow you to create and edit the threat tree for your current assessment. Additionally, selecting "EXPORT" will prompt you to select how you want to export your threat matrix. You can chose either XLSX or PDF.

Scenarios are able to be re-ordered by clicking and dragging the Scenario element and releasing somewhere else in the list of Scenarios. This will determine the order that the Scenarios will be exported in the Threat Matrix Spreadsheet. Once the order is how you want it, click "SAVE CHANGES" to commit the ordering.

## Creating Events

![Events Page](readme-images/Events.png)

To create an Event, select the Assessment and then Scenario you want to add the Event to. Once on the Events page, clicking "CREATE EVENT" will present you the option of either importing an Event from one of the Steplates already created, or create a new event.

Events on this page can be re-ordered if needed by clicking and dragging an Event item in the list and dropping it where you want it. This will determine the ordering of the Events in that particular Scenario in the exported Threat Matrix Spreadsheet. Once you are happy with the order they are in, click "SAVE CHANGES" to commit your ordering changes.

From this page you may edit the Scenario title by clicking "EDIT SCENARIO", delete the current Scenario by clicking "DELETE SCENARIO" or return to the Scenarios page by clicking "SCENARIOS". You can also generate a scenario graph by clicking "EXPORT GRAPH".

## Editing an Event

![Edit Event Page](readme-images/EditEvent.png)

By selecting the "MAKE TEMPLATE" button, a new Template will be created using the current event and you will be taken to the Edit Template page to finalize any changes.

### NIST Description Helper

![NIST Descriptions](readme-images/NISTDescription.png)

While filling in the individual factors for the event you are editing, you can take advantage of the helper info buttons on most of the factors. Clicking on these will display a modal overlay that contains the NIST descriptions for each severity level. BLS also includes our own interpretations of the NIST descriptions to help reduce any potential confusion.

### MITRE ATT&CK IDs

![MITRE ATT&CK IDs](readme-images/MITREIDs.png)

You are able to select an appropriate MITRE ATT&CK ID by selecting the category which expands displaying the various MITRE ATT&CK Tactics for that category. Selecting one of the tactics then displays the various Techniques under that Tactic.

Each Technique has a helper icon you can click on that will take you to that Technique's detailed information page on the MITRE website in a new tab.

These Techniques will be present in the exported Threat Matrix as IDs hyperlinked to the MITRE ATT&CK website for your clients to use in their remediation efforts.

If the intention is to create a Threat Tree for an assessment, it is necessary to select a MITRE ATT&CK ID for each event desired in the final Threat Tree. The unique ATT&CK IDs are collected to form the various nodes available to the user while creating a tree.

### Threat Sources

![Threat Sources](readme-images/ThreatSources.png)

The threat sources presented in the list box are pulled from NIST guildlines on conducting a Risk Assessment.

### Relevance to Findings

![Relevance Factor](readme-images/Relevance.png)

Each Event should have some relevance to technical findings during your assessment.

### Automatic Risk Calculation

Based on your entries into this page, two factors are automatically calculated (Overall Likelihood and Risk) for you to avoid biasing outcomes.

#### How is Risk Calculated

NIST provides the information needed to properly calculate Risk. Three factors directly contribute to the final Risk value. These include Likelihood of Attack Initiation, Likelihood of Adverse Impact, and Level of Impact. The Likelihood factors are combined to form the Overall Likelihood value. Risk is then calculated based on combining the Overall Likelihood value and the Level of Impact value.

### Finding Reference/Vulnerability Severity and Predisposing Condition/Pervasiveness of Predisposing Condition

NIST provides guidance on quantifying a Serverity and Pervasiveness factor for a particular event. To make this more obvious and understandable, these have been broken out into separate fields. If a Finding Reference is not supplied, then the Vulnerability Severity slider will not be activated. Similarly, if a Predisposing Condition is not provided then the matching Pervasiveness of Predisposing Condition slider will be deactivated.

If only one of these are filled in, then the overall Severity and Pervasiveness factor in the exported Threat Matrix will be taken from the one enabled. If both are provided, then an average is taken of the two and floored.

### Graph Elements

![Graph Elements](readme-images/GraphChoices.png)

To create a graph export of your Scenarios, you will need to provide information for each event.

#### Entity

Which Entity you choose here will determine the graph node image displayed in the exported graph.

#### Description

The Description you provide will become the label underneath the graph image in the exported graph.

#### Preceded By

This list contains each event already present in the current Scenario. If there are no events that precede this one, or you want this particular event to be tied directly to the root node (The attacker node) then you can select None (Root Node). Otherwise, selecting on of the other events in the list will make that event the parent to the one you are editing currently.

It is also possible to have multiple preceded by values. This gives the user the ability to make more complex graphs showing multiple events leading to a single consequence, or singular events leading to multiple simultaneuos consequences.

## Creating a Template

![Templates Page](readme-images/Templates.png)

To create a Template, navigate to the Templates page. Once there, you can click "CREATE TEMPLATE". This will create a new blank Template and bring you to the Template editing page.

On this page, there is also an "IMPORT" button which will prompt you to select a proper template pack JSON file. Uploading the file will generate templates from the contents. To view the proper structure of the JSON file, it is recommended to first create a few Templates using the GUI and then export them using the "EXPORT" button.

The "DELETE ALL" button will prompt you to confirm you want to remove all templates from the database. Once confirmed they are removed.

## Editing a Template

![Edit Template Page](readme-images/EditTemplate.png)

Editing a Template is an identical process to editing an Event, however certain considerations should likely be made to make these as reusable as possible. This should allow you to maintain more consistency in quantifying event factors.

## Exporting Graphs

![Exported Graph](readme-images/Graph.png)

To export a threat graph for a specific Scenario, select the Assessment that the Scenario belongs to. From the Scenarios page, select the Scenario you want to export as a graph. From the Events page, click the "EXPORT GRAPH" button. This generates a graph using GraphViz in the background, and displays the resulting graph in the browser. The graph will have a transparent background once saved so it may be used in other reporting materials as supportive information. Also included is a simple markdown table that is populated with the event descriptions the user provided. These descriptions give the graph edge's more context. The table is written in markdown for easy inclusion with reports generated using [WriteHat](https://github.com/blacklanternsecurity/writehat).

To return to the application just use your browser's back button or click the "BACK" button.

## Exporting Threat Matrix Spreadsheet

![Exported Threat Matrix](readme-images/ThreatMatrix.png)

To export your Assessment to a Threat Matrix Spreadsheet, select the Assessment you want to export. Then, from the Scenarios page, click "EXPORT" followed by "XLSX". The default title for the exported spreadsheet will be AssessmentTitle_ThreatMatrix_YYYY_MM_DD. Exported XSLX documents are "living". This means that as the document is updated following remediation/mitigation efforts, the values for Overall Likelihood and Risk will be automatically adjusted to reflect those efforts.

## Exporting Threat Matrix PDF

![Exported Threat Matrix](readme-images/ThreatMatrixPDF-1.png)

To export your Assessment to a printable PDF, select the Assessment you want to export. Then, from the Scenarios page, click "EXPORT" followed by "HTML". This will present with a threat matrix formated to fit within tables that can be printed as a PDF. It is paginated to avoid page breaks within events, and is designed to be printed in landscape mode. To get the full effect of the page styling in the printed version, in your print settings, please disable headers and footers. Also, enable background graphics.

![Exported Threat Matrix](readme-images/ThreatMatrixPDF-2.png)

## Creating Threat Trees

![Threat Tree Creation](readme-images/ThreatTreeCreateNew.png)

When creating a threat tree for the first time under an assessment, the user is presented with the ability to create categories that will be used within the threat tree. These can be entirely custom so that the tree can convey whatever sort of information is necessary. Each category is assigned a color by the user that will be used to colorize that category's nodes.

![Threat Tree Editing](readme-images/ThreatTreeEdit-1.png)

Once the categories are entered and the tree is generated, the user is brought to the edit page. Here, the user has the options to save the current changes, delete the tree and start over, or go back to the scenarios screen. Clicking the "EXPORT" button will generate the threat tree graph along with a legend that can be used in other reporting utilities.

Each graph has a collection of settings that will determine how the resulting tree looks. Each settings has an info helper button to aid the end user in making selections. These info helpers will have either textual or graphical examples to ensure operator understanding.

![Threat Tree Editing](readme-images/ThreatTreeEdit-2.png)

The nodes in a threat tree are the set of MITRE ATT&CK IDs used across the entire assessment. Each node within a threat tree comes with its own collection of settings. Where these settings are not immmediately obvious, a info helper button has been included to provide more insight.

The section titled "How did this technique present itself?" is where the user would place the specific actions taken during the assessment that were categorized by the corresponding ATT&CK ID.

The section titled "Which techniques lead to TXXXX?" is where the edges are connected between nodes. It is possible to link multiple nodes to each other showing multiple paths of compromise.

![Threat Tree Export](readme-images/ThreatTreeExport.png)

Once the user finishes applying the desired settings, an exported threat tree might look similar to the above example.

# Methodologies

From our internal operations, we have functionally defined three methods for creating a threat narrative or scenario. They are listed here for consideration and discussion purposes. To understand the divergence of Simple/Dynamic Threat Source methodology, you will need to familiarize with [NIST SP-800-30r1](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-30r1.pdf):

![Description of Threat Sources](readme-images/methodology-1.PNG)
![Threat Sources](readme-images/methodology-2.PNG)
![Threat Source Characteristics](readme-images/methodology-3.PNG)

The vagueness of the definitions provided by NIST allow for an organization to interpret the information in whatever way best suits the organization within certain limits. 

## Simple Threat Source

The logical order for creating a scenario in the simplest way for less-seasoned operators is as follows:

1. Develop what the scenario is
   - Each event in the scenario should be able to be tied to a MITRE ATT&CK Technique
      - There are some exceptions. In these cases, it is helpful to bundle two events together
      - Example of exception:
         - `<impact>`: Attacker takes out 'shorts' against company stock (No MITRE coverage)
      - Example of how to bundle:
         - `<impact>`: Attacker leaks internal corporate financials and benefits financially by taking out 'shorts' against company stock ([Exfiltration Over C2 Channel - T1041](https://attack.mitre.org/techniques/T1041/))
   - Begin with `<initial access>`
   - `<Intermediate steps>`...
   - End with `<impact>`
1. Choose your threat source
   1. Who is this?
      - Individual
      - Group
      - Nation State/State Sponsored
   1. What are their capabilities? (C)
      - Individuals typically have a `[0-4]`
      - Groups typically have a `[5-7]`
      - Nation State/State Sponsored typically have a `[8-10]`
      - Capabilities consit of
         - Financing: Are they financed
         - Knowledge: What is their background/training
         - Infrastructure: Physical assets (cloud, server farms, computing power)
         - Time (Man-power, number of able bodies)
   1. What are their intents? (I)
      - This largely depends on how you are personifying your threat source
         - Are they motivated by boredom, the lulz, taking out aggression on the world and aren't really concerned about being detected? `[0-4]`
         - Are they motivated by financial gains and care a little about being detected? `[5-7]`
         - Are they very specific in what they want (incriminating evidence, leverage over a person) and are very careful about being detected? `[8-10]`
         - Some intents:
            - Financial Gain
            - Defacement, Destruction, Boredom
            - Political Activism
   1. How do they choose targets? (T)
      - Randomly or broadly scaled typically falls within `[0-4]`
         - Spray and pray, hot new CVE, mass phishing campaign using OSINT
      - Certain market verticals would typically be between `[5-7]`
         - They target banks/financial institutions using OSINT
      - Certain organization would be between `[8-10]`
         - They only target a certain corporation/organization using OSINT and information from past attacks
1. Now you should have the following items filled in all events
   - MITRE ATT&CK Technique
   - Event Description
   - Threat Source
      - Capability
      - Intent
      - Targeting
1. The next step is to decide how likely your threat source is to execute your scenario.
   - It is important to understand that any threat source can be placed in a scenario. 
   - The likelihood of them attempting that scenario should change based on the threat source and the scenario. 
   - This will affect the overall risk of what you are constructing.
   - Some examples:
      - A low C:I:T individual is unlikely to ninja their way into a government facility to thermite critical infrastructure
      - A high C:I:T nation state is unlikely to make itself known just to cause a small network outage at a random corporation 
      - A high C:I:T state sponsored organization is very likely to execute a carefully thought out attack to exfiltrate intellectual property
      - A low C:I:T organized group is very likely to deface political organizations based on ideology
   - You should now have Likelihood for Attack Initiation filled out for all events in the scenario
1. At this point, you have taken care of the factors that should be consistent across the entire scenario.
   - The following steps will need to be done while taking into consideration each particular event
1. Event Relevance
   - Apply the value that most fits the descriptions in ETM
1. Finding Reference
   - If there was a finding or findings discovered that is/are being reported on and it/they pertain/s to this event directly
   - Vulnerability Severity
      - This is only used if there is a Finding Reference
      - This should mirror the CVSS score of the finding, for example:
         - Finding Reference: `[T001]` Big-Bad-CVE (had a CVSS of 9.8)
         - Vulnerability Severity: 9
      - In the case of multiple findings, use the most sever as the severity value
1. Predisposing Condition
   - This is something that is not necessarily technical, but is considered an issue
   - Pervasiveness of Predisposing Conditions
      - This is only used if there is a Predisposing Condition
      - This should mirror the DREAD score or use best judgement to how widespread the issue is
         - Guidance is given within ETM
   - Example
      - Predisposing Condition: IT/Admin workstations were found unlocked and signed in during the assessment
      - Pervasiveness of Predisposing Conditions: This could be a company wide policy enforcement or training issue (High `[8-9]`)
1. Mitigation Observed
   - Can be one or more mitigations that directly affected or would affect the success of the event from happening
   - Should be considered Proactive Findings
1. Likelihood of Adverse Impact
   - You will need to take into account the previous 4 factors (Mitigation, Predisposing Conditions, Finding Reference, Event Relevance)
   - Based on those items, how likely is the adverse impact of this event to happen, however severe it is?
   - Example:
      - Event Description: Attacker lands C2 shell via phish
      - Finding Reference: `[None]`
      - Predisposing Condition: `[None]`
      - Mitigation Observed: Advanced email filtering (emails did not get through), Advanced EDR (binaries were blocked from executing), Advanced GPO (macros disabled in envrionment)
      - Likelihood of Adverse Impact: Low (There is always a chance of bypassing all these protections, but it is not likely)
1. Overall Likelihood
   - This is determined for you
   - It is calculated based on the Likelihood of Attack Initiation and Likelihood of Adverse Impact
1. Level of Impact
   - This should be determined based on the event outcome
   - You should **not** take into account any of the previous factors that you used when determining adverse impact likelihood
   - This is solely describing the potential impact of the event occuring
   - Example
      - Event Description: Attacker deploys ransomware to entire corporate domain
      - Level of Impact: High (unlikely to destroy a business or kill someone, but still pretty bad)
1. Risk
   - This is determined for you
   - It is calculated based on the Overall Likelihood and Level of Impact
   - It is important to understand that just because something his a high Level of Impact, it does not mean that the Risk is high.
   - If there is a moderate chance of the attacker executing the scenario, but there are many proactive controls in place, even a high impact event is considered low risk
   - Example
      - Likelihood of Attack Initiation: 5
      - Likelihood of Adverse Impact: 2
      - Overall Likelihood (Calculated): Low
      - Level of Impact: 9
      - Risk (Calculated): Low

## Dynamic Threat Source

This follows in much the same way as the Simple Threat Source methodology, except the idea is that the threat source's characteristics (C:I:T) evolve as the scenario plays out. Capability and Intent would be largely static, however may change in certain edge cases. Targeting would change as "threads are pulled" within the victim environment. Additionally, the likelihood for attack initiation would depend on the event and the current C:I:T of the threat source.

## `<Insert Threat Source Methodology>` and Sophisticated Intel

Uses either Simple or Dynamic Threat Source concepts. Additionally, takes into account information from threat intel sources including tracked adversaries (groups, APTs, etc), their C:I:T, and the TTPs they have been observed using. This would affect C:I:T (either throughout the scenario `[Simple]`, or as the base value `[Dynamic]`), and the likelihood for attack initiation. In this case, likelihood for attack initiation would change depending on whether the threat source had been observed performing this particular TTP, or a similar TTP. Also, likelihood for attack initiation would be based on the target organization and the adversary's typical targets of choice.

# References

* https://github.com/blacklanternsecurity/writehat
* https://csrc.nist.gov/News/2012/NIST-Special-Publication-800-30-Revision-1
* https://attack.mitre.org/
