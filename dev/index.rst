==========================
 向 Ceph 贡献：开发者指南
==========================

:作者: Loic Dachary
:作者: Nathan Cutler
:许可证: Creative Commons Attribution-ShareAlike (CC BY-SA)

.. note:: 旧的 (pre-2016) 开发者文档已经挪到了 \
   :doc:`/dev/index-old` 。

.. contents::
   :depth: 3


简介
====

This guide has two aims. First, it should lower the barrier to entry for
software developers who wish to get involved in the Ceph project. Second,
it should serve as a reference for Ceph developers.

We assume that readers are already familiar with Ceph (the distributed
object store and file system designed to provide excellent performance,
reliability and scalability). If not, please refer to the `project website`_ 
and especially the `publications list`_.

.. _`project website`: http://ceph.com 
.. _`publications list`: https://ceph.com/resources/publications/

Since this document is to be consumed by developers, who are assumed to
have Internet access, topics covered elsewhere, either within the Ceph
documentation or elsewhere on the web, are treated by linking. If you
notice that a link is broken or if you know of a better link, please
`report it as a bug`_.

.. _`report it as a bug`: http://tracker.ceph.com/projects/ceph/issues/new


必备知识
========

本章包含必要信息，每个 Ceph 开发者都应该知道。

项目领袖
--------

Ceph 项目是由 Sage Weil 领导的。另外，各主要项目组件有自己的领\
导，下面的表格罗列了所有领导、以及他们在 `GitHub`_ 上的昵称。

.. _github: https://github.com/

========= =============== =============
Scope     Lead            GitHub nick
========= =============== =============
Ceph      Sage Weil       liewegas
RADOS     Samuel Just     athanatos
RGW       Yehuda Sadeh    yehudasa
RBD       Josh Durgin     jdurgin
CephFS    Gregory Farnum  gregsfortytwo
Build/Ops Ken Dreyer      ktdreyer
========= =============== =============

上述表格里的 Ceph 专有缩写在下面的\ `体系架构`_\ 一节解释。

历史
----

请翻阅 `Wikipedia 的 History 这章`_\ 。

.. _`Wikipedia 的 History 这章`: https://en.wikipedia.org/wiki/Ceph_%28software%29#History

软件许可
--------

Ceph 是自由软件。

Unless stated otherwise, the Ceph source code is distributed under the terms of
the LGPL2.1. For full details, see `the file COPYING in the top-level
directory of the source-code tree`_.

.. _`the file COPYING in the top-level directory of the source-code tree`: 
  https://github.com/ceph/ceph/blob/master/COPYING

源代码仓库
----------

The source code of Ceph lives on `GitHub`_ in a number of repositories below
the `Ceph "organization"`_.

.. _`Ceph "organization"`: https://github.com/ceph

To make a meaningful contribution to the project as a developer, a working
knowledge of git_ is essential.

.. _git: https://git-scm.com/documentation

Although the `Ceph "organization"`_ includes several software repositories,
this document covers only one: https://github.com/ceph/ceph.

问题跟踪器
----------

Although `GitHub`_ is used for code, Ceph-related issues (Bugs, Features,
Backports, Documentation, etc.) are tracked at http://tracker.ceph.com,
which is powered by `Redmine`_.

.. _Redmine: http://www.redmine.org

The tracker has a Ceph project with a number of subprojects loosely
corresponding to the project components listed in `体系架构`_.

Mere `registration`_ in the tracker automatically grants permissions
sufficient to open new issues and comment on existing ones.

.. _registration: http://tracker.ceph.com/account/register

要报告软件缺陷或者提议新功能，请\ `跳转到 Ceph 项目`_\ 并点击 \
`New issue`_ 。

.. _`跳转到 Ceph 项目`: http://tracker.ceph.com/projects/ceph
.. _`New issue`: http://tracker.ceph.com/projects/ceph/issues/new

邮件列表
--------

Ceph 的开发邮件讨论是通过邮件列表 ``ceph-devel@vger.kernel.org`` \
进行的。这个邮件列表对所有人开放，把下面这行发送到 \
``majordomo@vger.kernel.org`` 即可订阅： ::

	subscribe ceph-devel

要作为邮件正文发出。

There are also `other Ceph-related mailing lists`_. 

.. _`other Ceph-related mailing lists`: https://ceph.com/resources/mailing-list-irc/

IRC
---

In addition to mailing lists, the Ceph community also communicates in real
time using `Internet Relay Chat`_.  

.. _`Internet Relay Chat`: http://www.irchelp.org/

See https://ceph.com/resources/mailing-list-irc/ for how to set up your IRC
client and a list of channels.

补丁的提交
----------

The canonical instructions for submitting patches are contained in the 
`the file CONTRIBUTING.rst in the top-level directory of the source-code
tree`_. There may be some overlap between this guide and that file.

.. _`the file CONTRIBUTING.rst in the top-level directory of the source-code tree`: 
  https://github.com/ceph/ceph/blob/master/CONTRIBUTING.rst

All newcomers are encouraged to read that file carefully.

从源码构建
----------

请参考 :doc:`/install/build-ceph` 。

开发模式集群
------------

编译完源码后，你可以启动一个开发模式的 Ceph 集群，命令如下：

.. code::

    cd src
    install -d -m0755 out dev/osd0
    ./vstart.sh -n -x -l
    # check that it's there
    ./ceph health


基本工作流
==========

.. epigraph::

    Without bugs, there would be no software, and without software, there would
    be no software developers. 

    --Unknown

    没有缺陷，就不会有软件；没有软件，就不会有软件开发者。

    ——无名

前面已经介绍了\ `问题跟踪器`_\ 和\ `源代码仓库`_\ ，也提及了\ \
`补丁的提交`_\ ，现在我们再详细解释一下它们在基本的 Ceph 开发流\
程里如何运作。

问题跟踪器惯例
--------------

When you start working on an existing issue, it's nice to let the other
developers know this - to avoid duplication of labor. Typically, this is
done by changing the :code:`Assignee` field (to yourself) and changing the
:code:`Status` to *In progress*. Newcomers to the Ceph community typically do not
have sufficient privileges to update these fields, however: they can
simply update the issue with a brief note.

.. table:: Meanings of some commonly used statuses

   ================ ===========================================
   Status           Meaning
   ================ ===========================================
   New              Initial status
   In Progress      Somebody is working on it
   Need Review      Pull request is open with a fix
   Pending Backport Fix has been merged, backport(s) pending
   Resolved         Fix and backports (if any) have been merged
   ================ ===========================================

拉取请求
--------

The Ceph source code is maintained in the `ceph/ceph repository` on
`GitHub`_.

.. _`ceph/ceph project on GitHub`: https://github.com/ceph/ceph

The `GitHub`_ web interface provides a key feature for contributing code
to the project: the *pull request*.

Newcomers who are uncertain how to use pull requests may read
`this GitHub pull request tutorial`_.

.. _`this GitHub pull request tutorial`: 
   https://help.github.com/articles/using-pull-requests/

For some ideas on what constitutes a "good" pull request, see
the `Git 提交的优良做法`_ article at the `OpenStack 项目百科`_.

.. _`Git 提交的优良做法`: https://wiki.openstack.org/wiki/GitCommitMessages
.. _`OpenStack 项目百科`: https://wiki.openstack.org/wiki/Main_Page


体系架构
========

Ceph is a collection of components built on top of RADOS and provide
services (RBD, RGW, CephFS) and APIs (S3, Swift, POSIX) for the user to
store and retrieve data.

See :doc:`/architecture` for an overview of Ceph architecture. The
following sections treat each of the major architectural components
in more detail, with links to code and tests.

.. FIXME The following are just stubs. These need to be developed into
   detailed descriptions of the various high-level components (RADOS, RGW,
   etc.) with breakdowns of their respective subcomponents.

.. FIXME Later, in the Testing chapter I would like to take another look
   at these components/subcomponents with a focus on how they are tested.

RADOS
-----

RADOS stands for "Reliable, Autonomic Distributed Object Store". In a Ceph
cluster, all data are stored in objects, and RADOS is the component responsible
for that. 

RADOS itself can be further broken down into Monitors, Object Storage Daemons
(OSDs), and client APIs (librados). Monitors and OSDs are introduced at
:doc:`/start/intro`. The client library is explained at
:doc:`/rados/api/index`.

RGW
---

RGW stands for RADOS Gateway. Using the embedded HTTP server civetweb_, RGW
provides a REST interface to RADOS objects.

.. _civetweb: https://github.com/civetweb/civetweb

A more thorough introduction to RGW can be found at :doc:`/radosgw/index`.

RBD
---

RBD stands for RADOS Block Device. It enables a Ceph cluster to store disk
images, and includes in-kernel code enabling RBD images to be mounted.

To delve further into RBD, see :doc:`/rbd/rbd`.

CephFS
------

CephFS is a distributed file system that enables a Ceph cluster to be used as a NAS.

File system metadata is managed by Meta Data Server (MDS) daemons. The Ceph
file system is explained in more detail at :doc:`/cephfs/index`.

.. WIP
.. ===
..
.. Building RPM packages
.. ---------------------
..
.. Ceph is regularly built and packaged for a number of major Linux
.. distributions. At the time of this writing, these included CentOS, Debian,
.. Fedora, openSUSE, and Ubuntu.

