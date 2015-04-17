---
layout: post
title:  Jira Cleaning
categories: jira
description: Couple of Python scripts I used to remove projects, user groups and permission schemes in Jira.
published: False
---

There is couple of things that can slow down Jira. The obvious issues would be [database access][jira-db], [disk access][jira-hd] or just not enough computing power. In my opinion, inappropriate use can be very harmful as well, and with older versions of Jira you could get significant performance boost just by "cleaning" old stuff.

To be honest, I am not sure how much speed can this cleaning bring with current version of Jira, or if at all, but it makes administration little bit easier.

# Removing old projects

In a big Jira instance you can end up with loads of projects that are not used anymore or that have not been updated for a long time. In our instance we used to have around 500 projects. Some of them were only for testing, some of them were merged, moved, or just left there.

To improve performance, we decided that we will remove all projects that were not updated for more than 2 years.

In order to do that I created MySQL query to get list of projects, so I could use it then in Python code.

Here is MySQL query to get projects that were last updated before 2013-01-01:

{% highlight mysql %}
SELECT
    p.id,
    p.pkey AS 'Project Key',
    p.pname AS 'Project Name',
    MAX(i.updated) AS 'Latest activity',
    COUNT(*) AS 'Number of Issues'
FROM
    jiraissue AS i,
    project AS p
WHERE
    i.project = p.id
GROUP BY p.id
HAVING MAX(i.updated) < '2013-01-01 00:00:00'
ORDER BY MAX(i.updated);
{% endhighlight %}

I stored output into the file. It should look similar to this:

{% highlight mysql %}
+-------+--------------+---------------+---------------------+------------------+
| id    | Project Key  | Project Name  | Latest activity     | Number of Issues |
+-------+--------------+---------------+---------------------+------------------+
| 10060 | PK1          | Project 1     | 2008-02-11 17:07:49 |                8 |
| 10022 | PK2          | Project 2     | 2008-03-06 09:23:02 |              135 |
...
{% endhighlight %}

Then I used Python code to remove projects. At first I thought that I can use [jira-python][jira-python] and `delete_project()` but that didn't work. So I had to use SOAP, while it works.

In my Python code, I do couple of things:

1. read output file from MySQL query
2. parse it to dictionary
3. iterating through project ids
4. finding project and deleting it

To run Python code:

{% highlight bash %}
$ python remove_projects.py [-test|-live] <file-with-projects.txt>
{% endhighlight %}

You can find all source code in [CleanJira/remove projects][clean-jira-projects] repository.

# Removing Jira User Groups

Removing user groups can speed up permission checking and then overall performance. In our instance we used to have around 200 user groups but later on we changed permission model and there was no need for that many groups.

To remove user groups I used the same procedure as with projects. First I get user groups from database.

{% highlight mysql %}
SELECT
    id, group_name
FROM
    cwd_group
GROUP BY group_name;
{% endhighlight %}

And result should look something like this.

{% highlight mysql %}
| id    | group_name      |
+-------+-----------------+
| 10148 | Test Group      |
| 10154 | ADVIS           |
| 10146 | ALI             |
...
{% endhighlight %}

In Python I used SOAP calls and method `deleteGroup()`. You can also specify not to remove certain groups, let say the groups which name starts with special word.

To run Python code:

{% highlight bash %}
$ python remove_usergroups.py [-test|-live] <file-with-usergroups.txt>
{% endhighlight %}

You can find all source code in [CleanJira/remove usergroups][clean-jira-usergroups] repository.

# Remove permission schemes without projects

After removing old projects you can end up with quiet a few permission schemes. You can easily delete them Jira administration section, but if number of schemes goes to hundred, why not write a script.

To get permission schemes without project, I wrote this query:

{% highlight mysql %}
SELECT
    *
FROM
    permissionscheme
WHERE
    id NOT IN (SELECT
            s.id
        FROM
            nodeassociation AS n,
            project AS p,
            permissionscheme AS s
        WHERE
            n.source_node_entity = 'Project'
                AND n.source_node_id = p.id
                AND n.sink_node_entity = 'PermissionScheme'
                AND n.sink_node_id = s.id);
{% endhighlight %}

Where result will look awfully familiar by now.

{% highlight mysql %}
| ID    | NAME        | DESCRIPTION  |
+-------+-------------+--------------+
| 10047 | ADM Tool    |              |
| 10260 | ALI         |              |
| 10374 | ALIE        |              |
| 10377 | ALIS        |              |
...
{% endhighlight %}

In Python I used SOAP calls again and method `deletePermissionScheme()`. This method is taking scheme name as argument, so it is important that it is included in MySQL query.

To run Python code:

{% highlight bash %}
$ python remove_permission_schemes.py [-test|-live] <file-with-permission-schemes.txt>
{% endhighlight %}

You can find all source code in [CleanJira/remove permission schemes][clean-jira-schemes] repository.

[jira-python]: http://pythonhosted.org//jira/
[clean-jira-projects]: https://github.com/ThePavolC/CleanJira/tree/master/remove%20projects
[clean-jira-usergroups]: https://github.com/ThePavolC/CleanJira/tree/master/remove%20usergroups
[clean-jira-schemes]: https://github.com/ThePavolC/CleanJira/tree/master/remove%20permission%20schemes
[jira-db]: https://confluence.atlassian.com/display/JIRAKB/Testing+Database+Access+Speed
[jira-hd]: https://confluence.atlassian.com/display/JIRAKB/Testing+Disk+Access+Speed
