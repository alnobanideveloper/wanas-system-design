# 9. Deployment and Infrastructure

| | |
| --- | --- |
| **Project** | Wanas (ونس) |
| **Document section** | arc42 — Section 9: Deployment and Infrastructure |
| **Status** | Draft for the final system |
| **Diagram** | Deployment |

This section places the final Wanas system on its host and describes how a release reaches that host. Seniors, volunteers, and family members use one React Native app. Administrators use a separate Next.js dashboard. The server side is one Docker Compose stack on one OVHcloud VPS-1.

Section 5 counts the admin dashboard, the API, and PostgreSQL as the server-side building blocks. Caddy stands in front of those three. It is the public edge of the same stack, and it is not another product module. The in-process worker stays inside the API container.

## 9.1 Where the system runs

Production is one OVHcloud VPS-1, as TC-01 requires. The published plan is 2 vCores, 4 GB RAM, 40 GB NVMe SSD, unlimited traffic, and a daily backup of the previous 24 hours. The public name is `wanas-jo.com` (TC-05). The datacenter is one of OVHcloud’s locations. The team administers Linux on that host (OC-03).

Docker Compose is the whole production runtime (TC-02, CV-13). The compose file is the description of what is running. Four containers share the machine:

| Container | Image role |
| --- | --- |
| Caddy | Terminates TLS and routes public HTTPS. |
| Admin dashboard | The Next.js application for administrators. |
| Wanas API | The C# ASP.NET Core monolith: the JSON API and the in-process worker. |
| PostgreSQL | The relational database (TC-03). |

Each container has one role (CV-07). The worker is a hosted service in the API process, so a release does not start a second worker container. There is no second machine and no managed database beside this VPS.

The React Native app is not installed on the VPS. Google Play and the App Store distribute it, and the phone runs it (TC-07, TC-12).

Two Docker volumes live on the 40 GB disk:

| Volume | Contents |
| --- | --- |
| PostgreSQL data | Accounts, matches, help, equipment, and reports. |
| Photo volume | Equipment-listing photos uploaded through the API. |

Replacing a container leaves both volumes in place (CV-10). The .NET image is built in stages, so the copy on the VPS holds the runtime and leaves the disk for that data (CV-09).

## 9.2 Deployment diagram

The editable source is [diagrams/deployment.puml](diagrams/deployment.puml).

![Wanas deployment diagram](diagrams/deployment.png)

The phone and the administrator’s browser sit outside the VPS. Google Play and the App Store install the app onto the phone. They do not run on the server. Public HTTPS stops at Caddy. Caddy routes the dashboard to the Next.js container and routes `https://wanas-jo.com/api` to the API. The API is the only container that opens PostgreSQL or the photo volume. WhatsApp, FCM, and APNs are called by that same API process.

## 9.3 What is deployed where

| Part | Where it runs | What it is for |
| --- | --- | --- |
| Wanas mobile app | The person’s phone, Android or iOS | Screens and local state for seniors, volunteers, and family members. Calls the API over HTTPS. Receives push on the phone. |
| Google Play and the App Store | Google and Apple | Publish the Android and iOS builds and install them on the phone. They are not containers on the VPS. |
| Caddy | Container on the VPS | TLS for `wanas-jo.com`. Routes the admin dashboard and `/api`. The only process that accepts public HTTPS. |
| Admin dashboard | Container on the VPS | Next.js application. Volunteer review, matches, reports, and listings, opened in the administrator’s browser. |
| Wanas API | Container on the VPS | C# monolith. JSON API, companionship rules, and the in-process worker for registration codes and reminders. |
| PostgreSQL | Container on the VPS | Accounts, matches, help, equipment, and reports. The API reaches it with SQL on the Docker network. The data files sit on the PostgreSQL volume. |
| Photo volume | Docker volume on the VPS disk | Equipment-listing photos. The API writes the files. The volume remains when the API container is replaced. |

## 9.4 Networking and external services

The VPS publishes the web ports and no database port (CV-11). Public HTTPS reaches only Caddy. PostgreSQL accepts connections only from the API container on the Docker network.

Caddy terminates TLS for `wanas-jo.com` (CV-14). On the Docker network it forwards the dashboard to the Next.js container over HTTP, and it forwards `https://wanas-jo.com/api` to the API over HTTP.

| From | To | Path |
| --- | --- | --- |
| Phone app | Caddy | JSON over HTTPS. |
| Administrator browser | Caddy | HTTPS for the dashboard, and JSON over HTTPS for `/api`. |
| Caddy | Admin dashboard | HTTP on the Docker network. |
| Caddy | Wanas API | HTTP on the Docker network, for `/api`. |
| Wanas API | PostgreSQL | SQL on the Docker network. |
| Wanas API | Photo volume | Listing photos on the Docker volume. |
| Wanas API | WhatsApp | HTTPS. A registration code. |
| Wanas API | FCM and APNs | HTTPS. A reminder. |
| FCM and APNs | Phone | Push to the installed app (TC-09). |
| Google Play and the App Store | Phone | Install of the mobile app. |

The phone loads OpenStreetMap tiles itself while a person places a home pin, as Section 5 describes. That request does not enter the VPS. The coordinates are saved later through the API.

## 9.5 Configuration and secrets

Container settings come from the environment (CV-12). Docker Compose on the VPS reads an env file when a container starts. That file is stored on the VPS. It is not copied into the image, and it is not committed to git (CV-05).

The file supplies:

| Secret | Used by |
| --- | --- |
| Database password and the API connection string | The API, to open PostgreSQL. PostgreSQL, to accept that login. |
| WhatsApp credential | The API, when the worker sends a registration code. |
| FCM credential | The API, when the worker sends an Android reminder. |
| APNs credential | The API, when the worker sends an iOS reminder. |

An image build stays free of machine-specific connection strings, the domain’s keys, and these credentials. The same pinned image can start on this host because the host supplies the values.

## 9.6 Deploy, update, and rollback

A release updates the Compose stack on the single VPS (CV-13). Production images use a pinned tag, so the running stack can be named from that tag (CV-08). The previous tag is kept on the host.

The API release does two steps, in this order (CV-15):

1. Entity Framework migrations run against PostgreSQL, so the schema matches the version that is about to serve traffic.
2. The new API image starts. It serves `https://wanas-jo.com/api`, and the worker starts inside that same process.

The admin dashboard is deployed the same way: the stack points at a new pinned Next.js tag, and the previous tag is kept. The dashboard and the API are on `wanas-jo.com` before a store release. Android and iOS builds are published against that API (CV-17).

Rollback starts the previous tag again from the same compose file (CV-16). That start does not replace the PostgreSQL volume or the photo volume. The data on disk remains the data those volumes already hold.

## 9.7 Logging, monitoring, and backup

Logs are the container logs of Caddy, the admin dashboard, the API, and PostgreSQL. A log line records the operation and the failure. It omits passwords, phone numbers, and home addresses (CV-04).

The host is one small VPS. Monitoring is the team reading those container logs and checking the machine: whether the four containers are running, and whether memory and the 40 GB disk still have room. The team does this on the host it already administers (OC-03).

The backup is OVHcloud’s daily backup of the VPS, which keeps the previous 24 hours (TC-01). The PostgreSQL volume and the photo volume are files on that machine, so they are part of the same backup. This daily backup is the backup approach. It is not a second backup product.

## 9.8 References

- Section 2, `02-architecture-constraints.md`, especially TC-01, TC-02, TC-03, TC-05, TC-07, TC-09, TC-12, OC-03, and CV-04 through CV-17.
- Section 4, `04-solution-strategy.md`, for the monolith, the in-process worker, and the single Compose stack on the VPS.
- Section 5, `05-building-block-view.md`, for the containers this deployment places on the phone and on the VPS.
- arc42 Section 9, Deployment View: https://docs.arc42.org/section-9/
