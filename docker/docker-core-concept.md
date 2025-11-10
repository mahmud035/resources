# What Docker Actually Is

Think of Docker as a way to package your entire application—your Node/Express backend, MongoDB database, React frontend, all the dependencies, environment variables, everything—into standardized "containers."

A container is like a lightweight, isolated box that has everything your app needs to run. It's not a full virtual machine (those are heavy and slow), but it's isolated enough that it runs the same way on your laptop, your teammate's Windows machine, and your production server.

# What is Images & Containers?

**Image = Class**
**Container = Instance of that class**

An image is the blueprint. A container is the running thing created from that blueprint.

You can create unlimited containers from one image, just like you can create unlimited objects from one class.

### What IS an Image?

An image is a **read-only template** that contains:
- The operating system (usually a minimal Linux)
- Your application code
- All dependencies (Node.js, npm packages, etc.)
- Environment configuration
- Instructions on how to run the app

Think of it as a snapshot—a frozen moment in time of everything your app needs.

### Images are Layered (This is Critical)

Here's what blows people's minds: Images aren't monolithic blobs. They're **stacked layers**, like a cake:

```
┌─────────────────────────────┐
│  CMD ["npm", "start"]       │  ← Layer 5: Startup command
├─────────────────────────────┤
│  COPY . .                   │  ← Layer 4: Your app code
├─────────────────────────────┤
│  RUN npm install            │  ← Layer 3: Dependencies
├─────────────────────────────┤
│  COPY package.json          │  ← Layer 2: Package file
├─────────────────────────────┤
│  FROM node:18-alpine        │  ← Layer 1: Base OS + Node
└─────────────────────────────┘
```
Each line in your Dockerfile creates a layer. Docker caches these layers.

Why this matters: If you change only your app code (layer 4), Docker doesn't rebuild layers 1-3. It reuses them from cache. This is why rebuilds are so fast.

**Important**: Stopping a container doesn't delete it. It's like pausing. You can start it again and it's exactly where you left off.

## Docker Containers: The Running Instance

### What IS a Container?

A container is a **running instance** of an image. It's the image brought to life.

Key characteristics:
- **Isolated**: Has its own filesystem, processes, network
- **Writable**: Unlike images, containers have a writable layer on top
- **Temporary**: When you delete a container, changes are gone (unless you use volumes)
- **Lightweight**: Shares the host OS kernel

# The Problem Volumes Solve
**Remember**: Containers are ephemeral. When you delete a container, everything inside it vanishes.

### Three Types of Data Persistence

Docker offers three ways to persist data:
```
┌─────────────────────────────────────────┐
│           Host Machine                  │
│                                         │
│  ┌──────────────────────────────────┐  │
│  │  Docker Volumes (managed by      │  │
│  │  Docker, best for production)    │  │
│  │  /var/lib/docker/volumes/        │  │
│  └──────────────────────────────────┘  │
│                                         │
│  ┌──────────────────────────────────┐  │
│  │  Bind Mounts (map host path      │  │
│  │  directly, best for development) │  │
│  │  /home/user/project → /app       │  │
│  └──────────────────────────────────┘  │
│                                         │
│  ┌──────────────────────────────────┐  │
│  │  tmpfs Mounts (in memory,        │  │
│  │  temporary, fast)                │  │
│  └──────────────────────────────────┘  │
└─────────────────────────────────────────┘
```
Focus on **volumes** and **bind mounts**. Those are 95% of what you'll use.

# Anonymous Volumes: The Troublemakers
Anonymous volumes are created without an explicit name. Docker generates a random hash.

# Named Volumes: Explicit and Manageable
Named volumes are volumes you explicitly name. They're trackable, reusable, and don't disappear mysteriously.

# The Mental Model
**Named volumes** = Your carefully labeled storage boxes
You know what's in each box, where it is, and can find it anytime.

**Anonymous volumes** = Mystery boxes in your basement
You have no idea what's in them, which are important, and they just accumulate until your basement is full.

**Be intentional with volumes**. Name them, track them, clean them up regularly. Your future self (and disk space) will thank you.

# What Are Bind Mounts? (Recap with Clarity)

**Volumes**: Docker manages storage in /var/lib/docker/volumes/. You reference by name.

**Bind Mounts**: You specify an exact path on your host machine that maps directly into the container.

```
Host Machine                    Container
┌─────────────────────┐        ┌──────────────┐
│ /home/mahmud035/    │        │              │
│   projects/         │   ←→   │  /app        │
│     ts-backend/     │ mount  │              │
└─────────────────────┘        └──────────────┘
```
Changes in `/home/mahmud035/projects/ts-backend/` **instantly appear** in `/app` inside the container. And vice versa.

# The Golden Rules
✅ Use bind mounts for development code (instant updates)
✅ Always protect node_modules with `-v /app/node_modules`
✅ Use volumes for databases, persistent data
✅ Run as your user to avoid permission issues
✅ Be selective with what you mount

❌ Don't bind mount databases (use volumes)
❌ Don't forget node_modules protection
❌ Don't hardcode absolute paths (use `$(pwd)`)
❌ Don't mount entire filesystem (security risk)

# What Are Docker Networks?
Docker networks are virtual network infrastructures that allow containers to communicate with each other and the outside world. 

**Without networks**: Containers are isolated islands, can't talk to each other.
**With networks**: Containers become a distributed system, communicating seamlessly.

# The Five Network Drivers
Docker provides different network drivers for different use cases:
┌─────────────────────────────────────────────────────────┐
│  1. bridge    - Default, single-host networking         │
│  2. host      - No isolation, use host's network        │
│  3. none      - No networking at all                    │
│  4. overlay   - Multi-host networking (Swarm/K8s)       │
│  5. macvlan   - Assign MAC addresses to containers      │
└─────────────────────────────────────────────────────────┘
We'll focus on **bridge** (the workhorse), touch on **host** and **none**, and mention overlay and macvlan for completeness.

# 1. Bridge Networks (The Default)
When you start Docker, it creates a default bridge network called `bridge`.

See It
docker network ls
```
```
NETWORK ID     NAME      DRIVER    SCOPE
abc123def456   bridge    bridge    local
def456ghi789   host      host      local
ghi789jkl012   none      null      local
```

**`bridge`** is the default network. Every container without a specified network goes here.

### How Bridge Networks Work

**Architecture:**
```
┌─────────────────────────────────────────────────────────┐
│                   Host Machine                          │
│                                                         │
│  ┌─────────────────────────────────────────────────┐  │
│  │          Docker Bridge Network                  │  │
│  │          (Virtual Switch)                       │  │
│  │                                                 │  │
│  │   ┌──────────┐  ┌──────────┐  ┌──────────┐    │  │
│  │   │Container1│  │Container2│  │Container3│    │  │
│  │   │172.17.0.2│  │172.17.0.3│  │172.17.0.4│    │  │
│  │   └──────────┘  └──────────┘  └──────────┘    │  │
│  │         ↓             ↓             ↓          │  │
│  │   ┌─────────────────────────────────────┐     │  │
│  │   │    docker0 (Virtual Bridge)         │     │  │
│  │   │    172.17.0.1                       │     │  │
│  │   └─────────────────────────────────────┘     │  │
│  └──────────────────────┬──────────────────────────┘  │
│                         │                             │
│                    ┌────┴────┐                        │
│                    │   NAT   │                        │
│                    └────┬────┘                        │
│                         │                             │
└─────────────────────────┼─────────────────────────────┘
                          │
                     Internet
 ```

# Key components:

1. **docker0 bridge**: Virtual network interface on host (like a virtual switch)
2. **Virtual ethernet pairs**: Each container gets a veth pair connecting it to the bridge
3. **Internal IP subnet**: Containers get IPs from a private range (172.17.0.0/16 by default)
4. **NAT**: Docker performs Network Address Translation for outbound traffic

### Network Design

**Three-tier architecture for security:**
```
┌─────────────────────────────────────────────────────────┐
│  frontend-network (Public Tier)                        │
│    - nginx                                             │
│    - React frontend                                    │
└────────────┬────────────────────────────────────────────┘
             │
┌────────────┴────────────────────────────────────────────┐
│  backend-network (Application Tier)                    │
│    - Express API (bridge between frontend and data)    │
└────────────┬────────────────────────────────────────────┘
             │
┌────────────┴────────────────────────────────────────────┐
│  data-network (Data Tier)                              │
│    - MongoDB                                           │
│    - Redis                                             │
└─────────────────────────────────────────────────────────┘
```

**Why three networks?**
- Frontend can't directly access database (security)
- Backend bridges frontend and data tiers
- Data tier is completely isolated from public access

## Dockerfile versus Compose file

A `Dockerfile` provides **instructions to build a container image** while a `Compose file` **defines your running containers**. Quite often, a Compose file references a Dockerfile to build an image to use for a particular service.
