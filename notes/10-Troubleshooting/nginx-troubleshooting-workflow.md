# Nginx Troubleshooting Workflow

## Purpose

This document records the basic troubleshooting order for Nginx.

## Workflow

Service -> Process -> Port -> HTTP -> Logs

## Reasoning

1. Check whether the service is managed and active.
2. Check whether Nginx processes exist.
3. Check whether the expected port is listening.
4. Check whether HTTP access is normal.
5. Check system logs and Nginx error logs.

## Key Principle

Follow the runtime chain first. Do not jump directly to configuration.

The runtime chain is:

Service exists? -> Process exists? -> Port exists? -> HTTP works? -> Logs explain why?

## Engineering Meaning

This workflow connects service management, process management, port listening, HTTP behavior, and log analysis.
