---
layout: post
title:  Jira Cleaning
categories: jira
description:
published: False
---

# Removing old projects

In a big Jira instance you can end up with loads of projects that are not used anymore or that have not been updated for a long time. In our instance we used to have around 500 projects. Some of them were only for testing, some of them were merged, moved, or just left there.

To improve performance, we decided that we will remove all projects that were not updated for more than 2 years.

In order to do that we created MySQL query to get list of projects, so we could verify it, and then use it in Python code.

MySQL queyr to get projects that were last updated before 2013-01-01:
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

We stored output into the file. It should look similar to this:

{% highlight mysql %}
+-------+--------------+---------------+---------------------+------------------+
| id    | Project Key  | Project Name  | Latest activity     | Number of Issues |
+-------+--------------+---------------+---------------------+------------------+
| 10060 | PK1          | Project 1     | 2008-02-11 17:07:49 |                8 |
| 10022 | PK2          | Project 2     | 2008-03-06 09:23:02 |              135 |
...
{% endhighlight %}

Then we used Python code to remove projects. We could use [jira-python][jira-python] and `delete_project()` but that doesn't work. So we can use SOAP, while it works.

In my Python code, I do couple of things:

1. read output file from MySQL query
2. parse it to dictionary
3. iterating through project ids
4. finding project and deleting it

You can find all source code in [CleanJira][clean-jira] repository.

[jira-python]: http://pythonhosted.org//jira/
[clean-jira]: https://github.com/ThePavolC/CleanJira
