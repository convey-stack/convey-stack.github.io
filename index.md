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
          url: /documentation
          external_url: false # Set to "false" if you're pointing to inner page
          style: filled # Choose style: "filled" or "bordered"
          icon: edit
        - label: Browse repository
          url: https://github.com/convey-stack
          external_url: true
          style: bordered
          icon: github

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
        description: Watch our free course about building microservices and take a look at other projects.
        buttons: # Add buttons below, there are examples with all available options
            - label: Distributed .NET Core
              url: https://devmentors.io/distributed-net-core/
              external_url: true # Set to "false" if you're pointing to inner page
              style: filled # Choose style: "filled" or "bordered"
              icon: # Choose from 266 icons in "Feather" icon set, list of all icons is available here - https://feathericons.com            
            - label: Chronicle
              url: https://github.com/chronicle-stack/chronicle
              external_url: true # Set to "false" if you're pointing to inner page
              style: bordered # Choose style: "filled" or "bordered"
              icon: # Choose from 266 icons in "Feather" icon set, list of all icons is available here - https://feathericons.com
            - label: Ntrada
              url: https://github.com/Ntrada/Ntrada
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