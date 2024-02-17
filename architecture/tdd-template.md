# TDD Template

**Creation Date:** Dec 17, 2022

**Status:** *<u>Draft</u>* → *Reviewable→* *Accepted* | *Abandoned*

| Author(s)           | Paola Alcantar       |                  |
|---------------------|----------------------|------------------|
| Owning Team         | Integrations Team    |                  |
| Reviewer(s)         | N/A                  | *Here have the name of the team reviewers* |
| Ticket (Git Number) | N/A                  | *Here have the name of the team approvals* |
| Deployment Approval | N/A                  | *Here include the name of the team approvals* |
| Pull Request        | N/A                  | *Here have the link of the PR*             |

## Problem Statement

Michi Store as of now, manage its HR operations (Accounts and Contact
details) manually between Workday and Salesforce, however, company
planned at the beginning of the year a growth of more than 25% on the
headcount by the end of the year, wherewith current process is not
doable since data operations like registers and modifications take long
to be performed and completed manually by the HR team.

During the year, cross-functional teams worked together to release
several data pipelines, including one from Workday to LDAP, in which
Workday passes data every hour to LDAP with the most recent updates on
the employee data.

Accounts and Contact data are used in recruiting, payroll, and HR
operations.

Mulesoft needs to be able to

-   Manage CRUD operations in Salesforce. That way HR team can leave behind the manual operation of accounts and contacts.

-   Manage insert/update operations in HRDW (MySQL)

-   Replicate data from LDAP to HRDW and Salesforce hourly.

-   Perform backfills and single operations over certain employees.

## Technical Integration Details

<table>
<colgroup>
<col/>
<col/>
</colgroup>
<thead>
<tr class="header">
<th>Runtime Version</th>
<th>4.4.0</th>
</tr>
<tr class="odd">
<th>Integration Type</th>
<th><p>Experience API</p>
<p>Process API</p>
<p>System API</p>
<p>Publish / Subscribe</p></th>
</tr>
<tr class="header">
<th>Cadence</th>
<th>1-hour events for Publish / Subscribe</th>
</tr>
<tr class="odd">
<th>Volume</th>
<th>5,000 per day</th>
</tr>
<tr class="header">
<th>Transaction Types</th>
<th>Insert, Update, Merge</th>
</tr>
<tr class="odd">
<th>Platforms</th>
<th>Anypoint Platform, Salesforce, LDAP, MySQL, ActiveMQ</th>
</tr>
<tr class="header">
<th>Integration method</th>
<th>REST API / Batch Process</th>
</tr>
<tr class="odd">
<th>Server(s)</th>
<th>On-prem (standalone server)</th>
</tr>
<tr class="header">
<th>CI/CD</th>
<th>Jenkins</th>
</tr>
<tr class="odd">
<th>Version Control System</th>
<th>Github</th>
</tr>
</thead>
<tbody>
</tbody>
</table>

## Data Storage

Information is stored in Salesforce and replicated into HRDW (MySQL)
hourly.

### Salesforce

<table>
<colgroup>
<col/>
<col/>
</colgroup>
<thead>
<tr class="header">
<th>User Name</th>
<th>paola.alcval@gmail.com</th>
</tr>
<tr class="odd">
<th>Password</th>
<th>Shared secretly</th>
</tr>
<tr class="header">
<th>Security Token</th>
<th>Shared secretly</th>
</tr>
<tr class="odd">
<th>Connection URL</th>
<th>https://login/test.salesforce.com/services/Soap/u/56.0</th>
</tr>
<tr class="header">
<th>Environment(s)</th>
<th>Prod, staging, dev</th>
</tr>
</thead>
<tbody>
</tbody>
</table>

### HRDW (MySQL)

<table>
<colgroup>
<col/>
<col/>
</colgroup>
<thead>
<tr class="header">
<th>Host</th>
<th>Localhost (Docker)</th>
</tr>
<tr class="odd">
<th>Database</th>
<th>hr</th>
</tr>
<tr class="header">
<th>User Name</th>
<th>root</th>
</tr>
<tr class="odd">
<th>Password</th>
<th>Shared secretly</th>
</tr>
<tr class="header">
<th>Environment(s)</th>
<th>Prod, staging, dev</th>
</tr>
</thead>
<tbody>
</tbody>
</table>

## Data Schemas

<table>
<colgroup>
<col/>
<col/>
<col/>
</colgroup>
<thead>
<tr class="header">
<th>LDAP Field</th>
<th>Salesforce Field</th>
<th>MySQL Field</th>
</tr>
<tr class="odd">
<th>employeeIdentificationNumber</th>
<th>Employee_number__c</th>
<th>emp_number</th>
</tr>
<tr class="header">
<th>cn</th>
<th>Ldap__c</th>
<th>emp_ldap</th>
</tr>
<tr class="odd">
<th>employeeEmail</th>
<th>Email</th>
<th>emp_email</th>
</tr>
<tr class="header">
<th>employeeName</th>
<th>Name</th>
<th>emp_full_name</th>
</tr>
<tr class="odd">
<th>employeeName</th>
<th>First_name__c</th>
<th>emp_first_name</th>
</tr>
<tr class="header">
<th>sn</th>
<th>Last_name__c</th>
<th>emp_last_name</th>
</tr>
<tr class="odd">
<th>telephoneNumber</th>
<th>Phone</th>
<th>emp_phone</th>
</tr>
<tr class="header">
<th>mobile</th>
<th>MobilePhone</th>
<th>emp_mobile_phone</th>
</tr>
<tr class="odd">
<th>address1<br />
address2<br />
zipCode<br />
city<br />
state<br />
county</th>
<th>MailingAddress</th>
<th>emp_address_1<br />
emp_address_2<br />
emp_zipcode<br />
emp_city<br />
emp_state<br />
emp_country</th>
</tr>
<tr class="header">
<th>birthdate</th>
<th>Birthdate</th>
<th>emp_birthday</th>
</tr>
<tr class="odd">
<th>employeeDepartmentNumber</th>
<th>Department</th>
<th>emp_department</th>
</tr>
<tr class="header">
<th>title</th>
<th>Title</th>
<th>emp_title</th>
</tr>
<tr class="odd">
<th>startDate</th>
<th>Start_date__c</th>
<th>emp_start_date</th>
</tr>
<tr class="header">
<th>endDate</th>
<th>End_date__c</th>
<th>emp_end_date</th>
</tr>
<tr class="odd">
<th>salary</th>
<th>Salary__c</th>
<th>emp_salary</th>
</tr>
<tr class="header">
<th>userType</th>
<th>Employee_type__c</th>
<th>emp_type</th>
</tr>
</thead>
<tbody>
</tbody>
</table>

## Proposed Solution

The proposed solution in this TDD correspond to

-   [<u>API Led Connectivity</u>](https://blogs.mulesoft.com/learn-apis/api-led-connectivity/what-is-api-led-connectivity/) provided by Mulesoft. The answer will involve the development of multiple applications that will support Contacts creation and make the infrastructure scalable to add new components on top of the development.

-   Publish / Subscribe, part of the event-driven design to manage the hourly replication from LDAP into Salesforce and HRDW.

## Network Diagram

[<u>Network Diagram from Lucidchart</u>](https://lucid.app/lucidchart/e4e98f90-7d40-4943-bd3a-a9b85d7251e3/edit?viewport_loc=-2045%2C-184%2C5120%2C1152%2C0_0&invitationId=inv_88254674-56fd-4677-9503-e855ce415590)

## Publish / Subscribe Diagram

[<u>Publish / Subscribe Diagram from Lucidchart</u>](https://lucid.app/lucidchart/1493a0f0-20f2-4307-bfb0-1315ab93dc61/edit?viewport_loc=-726%2C-144%2C3072%2C691%2C0_0&invitationId=inv_edbcf8d5-5cd3-4a0c-8883-521cfc831ffd)

## Architecture Diagram

[<u>Architecture Diagram from Lucidchart</u>](https://lucid.app/lucidchart/42163c04-f620-4a28-94e8-88a35c49a629/edit?viewport_loc=-707%2C207%2C5120%2C1152%2C0_0&invitationId=inv_b0fc5637-f828-44a8-9909-8078db08ee3d)

## Mule Applications

### Domain Project

A domain project will be created to provide reusability for ports, and
in common attributes on the HTTP transport.

### hr-experience-api

At the experience layer, the client will be able to query information
from a specific employee by LDAP, backfill data from LDAP into SFDC from
certain employees and will be able to search a bunch of employees by
sending a multipart/form-data file with the LDAPs to look or backfill.

### hr-process-api

This application will receive the request to create/update Contacts in
Salesforce using a binary file to specify the employees who will be
backfilled or synced. Any transformation would be placed in this
application.

### sfdc-system-api

This application will make the connection to the Salesforce platform. It
will allow one place to create, update and retrieve Contacts. This
project will be dependent on the domain project.

### ldap-system-api

This application will make the connection to the LDAP service. It will
allow one place to retrieve employee data and be used in the API Led
Connectivity and Pub/Sub approaches for reusability. This project will
be dependent on the domain project.

### ldap-producer

This application will be built as a publisher. The main task of this
application will be to grab any changes within the last hour from the
LDAP service and publish messages into a topic.

### ldap-sfdc-consumer

This application will be built as a consumer. The main task of this
application will be to consume the messages produced hourly by the
publisher and sync data into Salesforce.

### ldap-hrdw-consumer

This application will be built as a consumer. The main task of this
application will be to consume the messages produced hourly by the
publisher and sync data into HRDW.

## Logging

In this project, we will use the out-of-the-box logging configuration,
allowing the log4j2 to ingest logs as text files stored into the on-prem
standalone server. In the future request is to enhance logging ingesting
logs into Splunk or Kibana and generate reports for better monitoring.

## CI/CD

CI/CD pipelines will be managed by Jenkins using Maven and Anypoint CLI.

## Source Control

Source control will be managed using Github. This includes an Anypoint
Platform setup to link Design Center API specifications into a
repository in the Michi-org git organization.

## Error handling

Creating errors-library to standardize error response.

## Testing

-   UAT sessions with cross-functional teams.

-   Regression testing.

-   Automated testing using MUnits.

<table>
<colgroup>
<col/>
<col/>
<col/>
</colgroup>
<thead>
<tr class="header">
<th>Scenarios</th>
<th>Expected Results</th>
<th>Results</th>
</tr>
<tr class="odd">
<th>Replicate data from LDAP into Salesforce</th>
<th></th>
<th></th>
</tr>
<tr class="header">
<th>Replicate data from LDAP into HRDW</th>
<th></th>
<th></th>
</tr>
<tr class="odd">
<th>…</th>
<th></th>
<th></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

## Alternatives Considered / Steps in the future

-   Add health monitoring on the applications to improve visibility and observability on the applications. For reference: [<u>https://github.com/paola-alcval/health-monitoring-library</u>](https://github.com/paola-alcval/health-monitoring-library)

-   Improve logging and traceability by adding an error library and standardizing log messages.
