---
layout: post
title: "Podman for DevOps - Second Edition"
description: ""
category:
read: 11
tags: [linux, devops, container, podman]
comments: true
---

Having worked with [OpenStack](https://www.openstack.org/) and
[Ceph](https://ceph.io/) for many years, I've had a front-row seat to one of
the most significant transformations in modern infrastructure: the shift from
bare metal deployments to containerized workloads. When I started, everything
ran directly on physical machines. Then came virtual machines, and eventually,
containers changed everything.  
Before moving to [Red Hat OpenStack Services on OpenShift
(RHOSO)](https://docs.redhat.com/en/documentation/red_hat_openstack_services_on_openshift/18.0/),
I witnessed how [Podman](https://podman.io/) became the backbone of container
management in both OpenStack and Ceph deployments. Even today, it's still the
foundation for running nova-compute services on [EDPM (External Data Plane
Management)](https://openstack-k8s-operators.github.io/edpm-ansible/) nodes.  
Working with these systems in production has taught me how important it is to
understand the building blocks that power our container infrastructure.
That's why _"Podman for DevOps Second Edition"_ caught my attention.  
Authors Alessandro Arrichiello and Gianni Salinetti have produced something
that goes beyond typical technical documentation. They're tackling the question
I've been recently thinking about: what comes next in container technology,
especially with AI/ML workloads becoming mainstream?  

![Image\_hosted](/assets/images/posts/2026/podman.jpg)

The book addresses a gap I see all the time in the industry. Too many engineers
jump straight into [Kubernetes](https://kubernetes.io/) without understanding
what's actually running underneath. [Podman](https://podman.io/), along with
container runtimes like [CRI-O](https://cri-o.io/) and
[runc](https://github.com/opencontainers/runc), are the backbone of modern
infrastructure. You can't effectively troubleshoot complex issues, optimize
performance, or make architectural decisions at scale without understanding
these fundamentals.

##### Podman as the Foundation of Modern Container Infrastructure

Throughout the book, the authors make a compelling case that Podman and its
underlying runtime technologies represent the foundational layer upon which
modern container ecosystems are built. This perspective is particularly
important as the industry moves beyond the initial wave of container adoption
toward more sophisticated use cases involving AI, edge computing, and
security critical applications.  
The authors' exploration of Podman's daemonless architecture reveals why this
approach has become increasingly important for enterprise deployments. By
eliminating the traditional container daemon, Podman provides stronger security
boundaries and more predictable resource management, characteristics that
become essential when running AI workloads or managing containers in regulated
environments. The book demonstrates how this architectural decision creates
ripple effects throughout the entire container lifecycle, from image building
with [Buildah](https://buildah.io/) to image management with
[Skopeo](https://github.com/containers/skopeo).  
The coverage of [rootless
containers](https://docs.podman.io/en/latest/markdown/podman.1.html#rootless-mode)
and their integration with enterprise security frameworks like
[SELinux](https://selinuxproject.org/) showcases how Podman addresses
real-world deployment challenges. This section provides essential knowledge for
environments where security and auditability are not optional but fundamental
requirements for successful container deployment.

##### Integration with Modern DevOps Practices

I particularly appreciate the book's approach and its focus on how Podman
integrates with broader, well-established DevOps practices and emerging
technology trends. The authors provide concrete and valid examples to
demonstrate how this technology represents a solution for real-world
challenges.  
The discussion of container networking, storage, and security isn't presented in
isolation but as part of a cohesive approach to building reliable, scalable
infrastructure. This systems thinking is particularly evident in the chapters
covering integration with [systemd](https://systemd.io/) and Kubernetes, where
the authors show how Podman serves as a bridge between traditional system
administration and modern orchestration platforms.  
The book's exploration goes through image building and registry management with
Buildah and Skopeo, and it provides context for understanding how Podman fits
within the broader container ecosystem. Rather than treating these as separate
tools, the authors demonstrate how they work together to create secure,
efficient development and deployment pipelines (e.g.
[conmon](https://github.com/containers/conmon), the container monitoring
utility that manages container processes).  The coverage of troubleshooting and
monitoring techniques reflects a mature understanding of operational
requirements in production environments. The authors don't just explain how to
solve problems, they provide solutions and the right mindset for preventing
them or detecting them early.

##### The Revolutionary Impact of Podman AI Lab

The introduction of [Podman AI
Lab](https://github.com/containers/podman-desktop-extension-ai-lab) represents
a significant improvement that strategically aligns Podman with the current AI
world's infrastructure needs. The current approaches to running AI models are
largely based on cloud deployments with complex dependency chains and external
service requirements. Podman AI Lab demonstrates how AI workloads can run
entirely within local containerized environments.  
In an era where data sovereignty and privacy regulations are increasingly
important, the ability to run powerful AI models locally while maintaining full
control over data pipelines represents a competitive advantage that many
organizations might need. The authors walk readers through deploying
[OpenAI](https://openai.com/) compatible inference servers that eliminate
dependencies on external services.  
What really impressed me about the AI integration is the focus on real-world
implementation details. The authors demonstrate its use through concrete
examples that integrate [Large Language
Models](https://en.wikipedia.org/wiki/Large_language_model) into practical
applications. This approach makes advanced AI capabilities accessible to DevOps
teams who may not have deep machine learning expertise but understand the
operational requirements of production systems.  
The innovation extends to how AI models are managed and scaled within the
Podman ecosystem. The integration with emerging projects like
[Ramalama](https://github.com/containers/ramalama), a new tool for simplified
AI model management, demonstrates how the broader container ecosystem is
evolving to support AI workloads with purpose-built tooling. This pattern
positions containers as the natural platform for the next generation of
intelligent applications.

##### Quadlets and the Evolution of Declarative Container Management

While Podman AI Lab gets the attention for its AI capabilities, what I really
appreciated was the book's coverage of Quadlets.  
While it remains a less popular topic, Quadlets address a fundamental challenge
because it fills the gap between container native tooling and traditional system
administration practices.  
The authors demonstrate how
[Quadlets](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html)
bridge this divide by providing a declarative approach to container lifecycle
management that integrates seamlessly with systemd.  
This enables organizations to adopt container technologies without abandoning
their existing operational knowledge and tooling.  
The practical examples provided throughout the book show how Quadlets transform
complex multi-container deployments into manageable, declarative
configurations. The authors walk through scenarios like setting up complete
[GitLab](https://about.gitlab.com/) environments with
[MariaDB](https://mariadb.org/) backends, demonstrating how services that would
traditionally require extensive documentation and manual intervention can be
expressed as simple, reproducible configurations.  
What makes Quadlets particularly innovative is how they enable a gradual
transition from imperative to declarative infrastructure management.
You can still keep using familiar container commands while automatically
generating the systemd configurations.  
The integration with tools like [Podlet](https://github.com/containers/podlet)
further amplifies the power of the Quadlet approach. By automatically
generating Quadlet configurations from standard Podman commands, Podlet
eliminates the learning curve that might otherwise prevent teams from adopting
declarative container management.

##### Advanced Troubleshooting: A Foundation for Complex Environments

The troubleshooting methodologies presented in the book represent one of its
most valuable contributions, particularly the detailed exploration of
[nsenter](https://man7.org/linux/man-pages/man1/nsenter.1.html) for
namespace-level debugging. While the authors demonstrate these techniques in
the context of Podman, the underlying principles and skills translate directly
to more complex orchestrated environments, including Kubernetes clusters where
traditional debugging approaches often fall short.  
The book shows how powerful nsenter can be for attaching debugging sessions to
specific container namespaces while retaining access to host-level tools. This
approach becomes critical when dealing with minimal container images that
lack debugging utilities, a common scenario in production environments focused
on security and image size optimization.  
The authors demonstrate practical scenarios, such as diagnosing network
connectivity issues in database clients returning HTTP errors, using tools like
[tcpdump](https://www.tcpdump.org/) and [dig](https://linux.die.net/man/1/dig)
within container network namespaces.  
What makes this coverage particularly valuable is how these same techniques
apply to Kubernetes troubleshooting. In Kubernetes environments, pods often
lack shell access or necessary diagnostic tools, and logs may not provide
sufficient information for complex network or resource issues. The nsenter
techniques demonstrated in this book become essential skills for platform
engineers and SREs working with production Kubernetes clusters.  
When troubleshooting network traffic from pods, for example, engineers can rely
on nsenter to enter a Pod's network namespace and run tcpdump to capture and
analyze packet flows, a technique documented in various troubleshooting guides
(e.g., [Red Hat Knowledge Base articles](https://access.redhat.com/solutions/1611883)).  
The book's emphasis on understanding [namespace
isolation](https://man7.org/linux/man-pages/man7/namespaces.7.html),
[cgroups](https://www.kernel.org/doc/Documentation/cgroup-v1/cgroups.txt), and
[user namespaces](https://man7.org/linux/man-pages/man7/user_namespaces.7.html),
as well as the historical excursus, provides a solid foundation necessary
for effective troubleshooting in complex scenarios.  
The related Linux primitives are the common base for all the container
technologies. Understanding how isolation works, how network namespaces
provide separate networking stacks, how mount namespaces isolate filesystem
views, and how process isolation works, is critical when debugging issues that
span multiple layers of the container stack.  
These troubleshooting techniques really show you why it's so important to
understand how container runtimes actually work. Whether you're working with
Podman, CRI-O, or [containerd](https://containerd.io/) in a Kubernetes
environment, the fundamental debugging approaches remain consistent. The book's
practical examples of using nsenter to troubleshoot database connectivity
issues or diagnose SELinux context problems provide a general approach that can
be adapted to virtually any containerized environment.

##### Assessment and Recommendations

_"Podman for DevOps Second Edition"_ succeeds in accomplishing something that
many technical books attempt: it provides both immediate practical guidance and
long-term strategic insights. It can serve multiple audiences simultaneously,
from DevOps engineers learning and looking how to implement specific solutions
to more advanced users that are trying to understand how container technologies
are evolving.  
What I find particularly valuable is how this book bridges the gap between
foundational knowledge and high-level containerized application design.
In my experience, understanding runtime technologies becomes essential when
you're working at scale.  
The book's greatest strength lies in the recognition that technology adoption
is not just about learning new tools, but it's about understanding how those
tools fit into broader organizational objectives and industry trends.
The focus on AI integration, declarative management, and enterprise security
concerns reflects a deep understanding of where the industry is heading and
what challenges organizations will face as they evolve their infrastructure
practices.  
For teams currently using [Docker](https://www.docker.com/) and considering
migration to Podman, this book provides essential guidance not just on the
technical aspects of migration but on the strategic benefits that justify the
effort.  
For organizations planning their container strategies, the book provides
valuable insights into how container technologies are evolving to support
emerging workloads and operational requirements. The discussion of AI
integration, in particular, offers a preview of how containers will serve as
the foundation for the next generation of intelligent applications.

##### Final Thoughts

In an industry that often focuses on the latest trends without considering
foundational principles, _"Podman for DevOps Second Edition"_ provides a
refreshing perspective on how solid engineering decisions create platforms for
innovation. The authors have demonstrated that by focusing on fundamental
architectural strengths like security, simplicity, and integration with
existing systems, it's possible to build container platforms that enable rather
than constrain emerging use cases.  
I'd recommend this [book](https://packt.link/f85LQ) to anyone who wants to truly
understand modern container infrastructure, not just use it. The container
ecosystem has become so fundamental to how we build and deploy applications
that superficial knowledge is no longer sufficient.  
For anyone responsible for container strategy, infrastructure architecture, or
DevOps implementation, this book provides essential insights into how container
technologies are evolving and what that evolution means for practical
deployment decisions.
