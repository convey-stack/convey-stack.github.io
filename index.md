---
# Page settings
layout: homepage # Choose layout: "default", "homepage" or "documentation-archive"
title: # Define a title of your page
description: # Define a description of your page
keywords: # Define keywords for search engines

# Hero section
hero:
    title: Convey
    text: A simple recipe for .NET Core microservices.
    background_image: # Paste image URL to display image in background of hero section
    buttons: # Add buttons below, there are examples with all available options
        - label: Getting started
          url: /documentation/getting-started/
          external_url: false # Set to "false" if you're pointing to inner page
          style: filled # Choose style: "filled" or "bordered"
          icon: edit
        - label: See it in action
          url: https://www.youtube.com/watch?v=cxEXx4UT1FI
          external_url: true
          style: bordered
          icon: youtube

# Features section
features:
    rows: # Add feature rows below, there are examples with all available options
        - title: What is Convey and what is not?
          description: 'Convey is a set of helper libraries that can be used independently of each other to help you to build your web applications and microservices, yet it is neither a framework nor a silver bullet. Convey does provide utilities to tackle challenges such as messaging, service discovery, load balancing, secure configuration, monitoring, distributed tracing and many more.'
          grid: # Add feature grid items below, there are examples with all available options
              - title: Lightweight
                description: Convey packages are very light both, in terms of size and external dependencies and can be used in any .NET Core application.
                icon: box
              - title: Modular
                description: Multiple packages can be used together to tackle different concerns as well as used independently of each other.
                icon: server
              - title: Simple
                description: Simplicity is the key to build small and reusable packages, thus you can easily fork our repositories and create the custom extensions.
                icon: cpu

    footer: # Set features section footer variables
        title: Other projects
        description: Watch our courses about building microservices and take a look at other projects.
        buttons: # Add buttons below, there are examples with all available options
            - label: Microservices .NET
              url: https://devmentors.io/courses/microservices-net
              external_url: true # Set to "false" if you're pointing to inner page
              style: filled # Choose style: "filled" or "bordered"
            - label: Distributed .NET Core
              url: https://www.youtube.com/watch?v=6YYB8vv3pZg&list=PLqqD43D6Mqz38LoZEuo_hJAp2NxXskcut
              external_url: true # Set to "false" if you're pointing to inner page
              style: bordered # Choose style: "filled" or "bordered"
              icon: # Choose from 266 icons in "Feather" icon set, list of all icons is available here - https://feathericons.com            
            - label: Pacco
              url: https://github.com/devmentors/Pacco
              external_url: true
              style: bordered
              icon:
            - label: Trill
              url: https://github.com/devmentors/Trill
              external_url: true
              style: bordered
              icon:
            - label: Chronicle
              url: https://github.com/snatch-dev/Chronicle
              external_url: true # Set to "false" if you're pointing to inner page
              style: bordered # Choose style: "filled" or "bordered"
              icon: # Choose from 266 icons in "Feather" icon set, list of all icons is available here - https://feathericons.com
            - label: Ntrada
              url: https://github.com/snatch-dev/Ntrada
              external_url: true
              style: bordered
              icon:
            - label: Pactify
              url: https://github.com/snatch-dev/Pactify
              external_url: true
              style: bordered
              icon:
---

        buttons: # Add buttons below, there are examples with all available options
            - label: Chronicle
              url: http://example.com
              external_url: true # Set to "false" if you're pointing to inner page
              style: filled # Choose style: "filled" or "bordered"
              icon: # Choose from 266 icons in "Feather" icon set, list of all icons is available here - https://feathericons.com
            - label: Ntrada
              url: /documentation
              external_url: false
              style: bordered
              icon:
---
