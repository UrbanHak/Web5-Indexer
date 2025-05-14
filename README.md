# Web5 Indexer

A simple open-source indexer and discovery engine for the Web5 ecosystem. Built to map and connect the emerging world of Bitcoin-native infrastructure across DePIN, DeFi, and Decentralized Telecom projects.

## Features
- GitHub-based project scraper and taxonomy matcher
- FastAPI backend with /projects endpoint
- Categorization via keywords into Web5 verticals
- Easy to extend, fork, and self-host

## Roadmap
- [x] Project taxonomy via YAML
- [x] GitHub + web scraper integration
- [x] REST API and caching
- [ ] Nostr or IPFS export
- [ ] Frontend dashboard (optional)

## License
MIT License © 2025 Jorge Cortes

# app/scraper.py
import requests
import json
import yaml

# taxonomy
with open("data/taxonomy.yaml") as f:
    taxonomy = yaml.safe_load(f)

def get_tags(description):
    tags = []
    if not description:
        return tags
    for category in taxonomy['categories']:
        for kw in category['keywords']:
            if kw in description.lower():
                tags.append(category['name'])
                break
    return list(set(tags))

def get_repos_from_org(org_name):
    url = f"https://api.github.com/orgs/{org_name}/repos"
    headers = {"Accept": "application/vnd.github+json"}
    response = requests.get(url, headers=headers)
    if response.status_code != 200:
        print(f"Failed to fetch repos for {org_name}")
        return []
    return response.json()

def scrape_all():
    orgs = ["start9labs", "fedibtc", "stacks-network", "holepunch-to"]
    all_projects = []

    for org in orgs:
        repos = get_repos_from_org(org)
        for repo in repos:
            project = {
                "name": repo['name'],
                "org": org,
                "description": repo['description'],
                "tags": get_tags(repo['description'] or ""),
                "url": repo['html_url'],
                "last_updated": repo['updated_at']
            }
            all_projects.append(project)

    with open("data/projects.json", "w") as f:
        json.dump(all_projects, f, indent=2)

if __name__ == "__main__":
    scrape_all()
