# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a personal CS learning notes repository managed with Obsidian. It contains study notes from various computer science courses and technical topics, primarily written in Chinese (中文).

## Content Structure

The repository is organized by course/topic directories:

- **University Courses**: CMU (15-213 CSAPP, 15-418 Parallel Computing, 15-445 Database, 11-667 LLM), MIT (6.824 Distributed Systems, 6.S081 OS), Stanford (CS106L C++, CS144 Networking), UCB (CS61A), Harvard (CS50 AI)
- **Technical Topics**: cuda, nccl, d2l (deep learning), quant (quantitative finance), kubernetes, docker, ebpf, golang, ansible, prometheus, elasticsearch, cmake, antlr, hpc, lustre
- **Each course directory typically contains**: `lecture.md`, `lab.md`, `syllabus.md`, and an `img/` folder for images

## Working with This Repository

### File Organization
- All notes are markdown files (.md) organized by topic/course
- Images are stored in `img/` subdirectories within each topic folder
- The `.obsidian/` directory contains Obsidian workspace configuration

### Language
- Primary language: Chinese (中文)
- When creating or editing notes, maintain consistency with existing Chinese content
- Technical terms may be in English with Chinese explanations

### Note Structure
- Course notes follow a consistent pattern: syllabus → lectures → labs
- Each markdown file uses standard markdown with Obsidian-compatible syntax
- Images are referenced using relative paths to local `img/` folders

### Git Workflow
- Main branch: `main`
- Commit messages should be concise and descriptive
- The repository tracks learning progress across multiple CS topics

## Obsidian Integration

This is an Obsidian vault. When editing:
- Maintain markdown compatibility with Obsidian
- Preserve internal links using `[[link]]` syntax if present
- Keep image references relative to maintain portability
