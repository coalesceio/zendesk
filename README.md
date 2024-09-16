<p align="center">
    <a alt="License"
        href="https://github.com/coalesceio/zendesk/blob/zendesk_views/LICENSE">
        <img src="https://img.shields.io/badge/License-Apache%202.0-blue.svg" /></a>
    <a alt="Maintained?">
        <img src="https://img.shields.io/badge/Maintained%3F-yes-green.svg" /></a>
    <a alt="PRs">
        <img src="https://img.shields.io/badge/Contributions-welcome-blueviolet" /></a>
    <a alt="Fivetran Quickstart Compatible"
</p>

# Zendesk Transformation Coalesce Pipeline ([Docs](https://github.com/coalesceio/zendesk))
# What does this Coalesce pipeline do?
- Produces modeled tables that leverage Zendesk data from [Fivetran's connector](https://fivetran.com/docs/connectors/applications/zendesk) in the format described by [this ERD](https://fivetran.com/docs/connectors/applications/zendesk#schemainformation).
- Enables you to better understand the workload, performance, and velocity of your team's work using zendesk issues. It performs the following actions:
  - Creates a daily issue history table so you can quickly create agile reports, such as burndown charts, along any issue field.
  - Enriches the core issue table with relevant data regarding its workflow and current state.
  - Aggregates bandwidth and issue velocity metrics along projects and users.
- Generates a comprehensive data dictionary of your source and modeled Zendesk data available in your Coalesce Organization [Environment or Generated Documentation](https://docs.coalesce.io/docs/generated-documentation).

<!--section="zendesk_transformation_model"-->
The following table provides a detailed list of all models materialized within this package by default. 

| **Model**                    | **Description**                                                                                                                                                 |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| v_final_ticket_metrics      | Each record represents a Zendesk Support ticket, enriched with metrics about reply times, resolution times, and work times.  Calendar and business hours are supported.  |
| v_final_ticket_enriched    | Each record represents a Zendesk Support ticket, enriched with data about its tags, assignees, requester, submitter, organization, and group.                           |
| v_final_ticket_summary       | A single record table containing Zendesk Support ticket and user summary metrics.                                                              |
| v_final_ticket_backlog           | A daily historical view of the ticket field values defined in the `ticket_field_history_columns` variable for all backlog tickets. Backlog tickets being defined as any ticket not in a 'closed', 'deleted', or 'solved' status.                                                             |
| v_final_ticket_field_history | A daily historical view of the ticket field values defined in the `ticket_field_history_columns` variable and the corresponding updater fields defined in the `ticket_field_history_updater_columns` variable.                                                        |
| v_final_sla_policies          | Each record represents an SLA policy event and additional sla breach and achievement metrics. Calendar and business hour SLA breaches are supported.    
<!--section-end-->

# How do I use the Coalesce pipeline?

## Step 1: Prerequisites
To use this Coalesce pipeline, you must have the following:

- A Fivetran zendesk connector syncing data into your **Snowflake** destination.
- Schemas that map to Coalesce Storage Locations
- Coalesce Markeplace Packages (already included in Git repo)
  - [Base Node Types](https://docs.coalesce.io/page/package_coalesce_base-node-types) (version 1.1.2+)
  - [Incremental Loading](https://docs.coalesce.io/page/package_coalesce_incremental-loading) (version 1.1.0+)
  - [Dynamic Tables](https://docs.coalesce.io/page/package_coalesce_dynamic-tables) (version 1.1.3+)

## Step 2: Import the coalesceio Zendesk repository

1. In Github select create a new repository and select `Import a repository`.
2. The URL for the Zendesk source repository is https://github.com/coalesceio/zendesk.git.  This is a Public repository so no credentials are required.
3. Select an owner and give your repository a name.
4. After the import is complete you will see one branche available:
    - **Zendesk_views** - This is an entire pipeline of views to manage the Zendesk pipeline.
5. This branch can be selected when setting up a Workspace which will be described below.

## Step 3: Set up a Project / Workspace in Coalesce

1. After the Git repo has been imported follow the Coalesce [documentation](https://docs.coalesce.io/docs/projects#create-a-new-project) to create a new project.  Initially, choose the option `Skip and Create` in the window for `Setup Version Control`.  We will connect to the Git repository after creating a Workspace.
Once the Project has been created select `Create Workspace`.  Enter a name and meaninful desription based on the Git branch you want to start from, either Dynamic Table or full load based.
2. At this point we are going to set up version control.  Select `Project Settings` and in the [Git Repository](https://docs.coalesce.io/docs/changing-a-git-repository-in-coalesce) section enter the URL of the repository you imported into your Git account as the Git Repository URL.
3. Save the `Project Settings`.
4. If you have enabled security for your Git repo, [Configure Git Account](https://docs.coalesce.io/docs/set-up-your-git-integration#add-through-the-project-dashboard).
5. After configuring the Git repo select `Launch` to launch the Workspace so we can attach it to a Git branch.
6. A Workspace can be attached to a branch by either selecting the `Git` modal or selecting `git branch` from the Workspace warning message `"Finish setting up version control for this workspace and avoid losing any work. Attach this workspace to a git branch"`.
7. After the `Attach Workspace to Branch` opens select the desired branch - **zendesk_views** to attach and `Attach` it.
8. Click on the `Git` modal, navigate to the `Branches` tab and select the [Branch Action](https://docs.coalesce.io/docs/git-branches#branch-actions) `Force Checkout` to populate the workspace with the latest Git commit.
9. This will overwrite any uncommitted work in the Workspace, which is what we want, so you will be required to confirm the Force Checkout by typing **FORCE** in the screen.
10. At this point the DAG objects should appear in your Workspace with errors.  Some workspace configuration is required to fix these errors.

## Step 4: Workspace Configuration

In this section you will configure the following Workspace settings:
- **Build Settings** - Configure [Storage Locations](https://docs.coalesce.io/docs/storage-locations-and-storage-mappings)
- **Workspace** - Configure Settings, User Credentials, Storage Mappings and Parameters

### Build Settings
Build Settings changes are required for the `zendesk_views` version of the Zendesk pipeline.

#### Storage Locations
The pipeline equires four Storage Locations be created.
- **Target (Default Location)** - The final output of the pipeline is stored in the `Target` location.  
- **SRC** - Source location where Fivetran replicated tables are locations

#### Environments
Environments must be configured in order to deploy pipeline to higher level environments (QA, UAT, Pre-Prod, Prod, etc.) based on how you are managing your environments.

If implementing the `zendesk_views` then no parameters are required.

### Workspace Settings

- **Settings** - Configure the Snowflake account that Coalesce will be utilizing
- **User Credentials / OAuth Settings** - Enter the credentials required to connect to Snowflake
- **Storage Mappings** - This can be configured to use one database / schema for all Storage Locations or up to four database / schema mappings, one for each Storage Location, depending on whether or not you want to seperate Source, Staging, Intermediate and Target objects.
- **Parameters** - Not required.

# Running / Deploying the Pipeline
The way the Zendesk pipeline is deployed and run differs between the two versions of the pipeline.

##  Views DAG Specifics
### Pipeline Definition
The pipeline is comprised of 17 sources, 17 incremental table, 17 persistent table, 5 work table and 166 views.  

### Missing Sources
If there are sources not available in your specific case (SPRINT, COMPONENT, VERSION), nodes related to those areas can be deleted from the pipeline.  This will require modification of any downstream objects that rely on said sources, but should be quick to figure out utilizing Coalesce object and column level lineage capabilities.

### Executing Pipeline

After the Workspace has been configured commit any changes into Git.  If the only changes have been Build Settings and Workspace Settings then there may be nothing to commit.

At this point you can create the pipeline.  The easiest way to do this is select `Create All` from the Graph action menu.

Assuming there are no errros the Zendesk Pipeline has now been deployed into your Development environment.  To refresh the pipeline simply run the node **zendesk_views_model_job** in the Dev Workspace.

From here you can deploy to higher level environments, assuming you have created environments, utilizing the standard Coalesce deployment mechanisms.

Once successfully deployed, execute the Job **zendesk_views_model_job** against an environment.

# How is this pipeline maintained and can I contribute?
## Pipeline Maintenance
The Coalesce team maintaining this package _only_ maintains the latest version of the package. 

## Contributions
A small team of analytics engineers at Coalesce develops these pipelines. However, the pipelines are made better by community contributions! 

# Are there any resources available?
- If you have questions or want to reach out for help, please refer to the [GitHub Issue](https://github.com/coalesceio/jira/issues) section to find the right avenue of support for you.
- If you would like to provide feedback to the Coalesce pipeline team or would like to request a new Coalesce pipeline, fill out our [Feedback Form](https:tbd).
- Any other questions about Coalesce then check out our [Help Center](https://help.coalesce.io/)
