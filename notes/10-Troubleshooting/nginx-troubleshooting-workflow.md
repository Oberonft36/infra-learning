# Nginx Troubleshooting Workflow

## Purpose

This document records the basic troubleshooting order for Nginx.

## Workflow

Service -> Process -> Config -> Port -> HTTP -> Logs

## Reasoning

1. Check whether the service is managed and active.
2. Check whether Nginx processes exist.
3. Check whether the configuration syntax is valid.
4. Check whether the expected port is listening.
5. Check whether HTTP access is normal.
6. Check system logs and Nginx error logs.

## Key Principle

Follow the runtime chain first. Do not jump directly to restart.

The runtime chain is:

Service exists? -> Process exists? -> Config is valid? -> Port exists? -> HTTP works? -> Logs explain why?

## Config Check

Use nginx configuration test before repeated restart attempts.

Command:

nginx -t

## Engineering Meaning

This workflow connects service management, process management, configuration validation, port listening, HTTP behavior, and log analysis.
