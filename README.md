# edu-python-webscraping

> Web scraping innebär att vi försöker läsa html sidan och hitta information i den.
>
> Syften för web scraping varierar och man bör följa användarvillkor för sajten man screjpar.
> Här följer några exempel ...
> 1. Dataanalys och forskning: För att samla in data för marknadsundersökningar,
> 2. akademisk forskning eller konkurrentanalys.
> 3. Övervakning av pris: För att spåra produktpriser över olika e-handelswebbplatser för prisjämförelser eller prisstrategier.
> 4. Ledgenerering: För att samla kontaktinformation från olika källor för försäljnings- och marknadsföringskampanjer.
> 5. SEO-övervakning: För att spåra sökmotorernas placeringar och optimera webbplatsers synlighet.
> 6. Sociala medier och sentimentanalys: För att samla in inlägg och kommentarer för att analysera allmänhetens uppfattningar.
> 7. Automatisering och integration: För att integrera information från flera källor in i interna system, som CRM-system eller databaser.
> 8. Innehållsaggregering: För att samla nyheter, blogginlägg eller artiklar från flera källor för innehållswebbplatser eller forskningsändamål.
> ...

## Prepare

> We use soup

https://pypi.org/project/beautifulsoup4/

```bash
cd ~
cd ws
mkdir web_scraping && cd web_scraping
pip install requests
pip install beautifulsoup4
```

## Create a search engine

> This is how early search 

## crawlsite

```python
cd ~
cd ws
cd web_scraping
echo '#!'"$(which python3)" >  crawlsite
chmod +x crawlsite
cat >> crawlsite << 'EOF'
import argparse
import requests
from bs4 import BeautifulSoup
from urllib.parse import urlparse, urljoin
import sys

def is_valid_url(base_url, url):
    """Check if a URL belongs to the same website as the base URL."""
    return urlparse(url).netloc == urlparse(base_url).netloc

def scrape_site(start_url, output=None):
    """Scrape the site for local URLs."""
    base_url = f"{urlparse(start_url).scheme}://{urlparse(start_url).netloc}"
    visited_urls = set()
    urls_to_visit = [start_url]

    # Only open the file once if output is specified.
    file = open(output, 'a') if output else None

    try:
        while urls_to_visit:
            current_url = urls_to_visit.pop(0)
            if current_url in visited_urls:
                continue

            visited_urls.add(current_url)
            try:
                response = requests.get(current_url)
                if response.headers.get('Content-Type', '').startswith('text/html'):
                    soup = BeautifulSoup(response.text, 'html.parser')
                    for link in soup.find_all('a', href=True):
                        href = link['href']
                        absolute_url = urljoin(current_url, href)
                        if is_valid_url(base_url, absolute_url) and absolute_url not in visited_urls:
                            if file:
                                file.write(f"{absolute_url}\n")
                            else:
                                print(absolute_url)
                            urls_to_visit.append(absolute_url)

                if output:
                    # Dynamically update the count of found URLs on the same line if output file is specified
                    sys.stdout.write(f"\rFound URLs: {len(visited_urls)}" + " " * 20)
                    sys.stdout.flush()

            except requests.RequestException as e:
                sys.stderr.write(f"\nRequest failed: {e}\n")
                sys.stderr.flush()

    finally:
        if file:
            file.close()
            print(f"\nURLs have been written to {output}")
        elif output:
            # Final count is printed on a new line after the loop completes if output file is specified.
            print(f"\rTotal Found URLs: {len(visited_urls)}".ljust(50))

def main():
    """Main function to parse arguments and control the script flow."""
    parser = argparse.ArgumentParser(description="Scrape a website for local URLs.")
    parser.add_argument("start_url", help="The starting URL to begin scraping from.")
    parser.add_argument("-o", "--output", help="Optional: Output file to write the URLs to continuously.", default=None)

    args = parser.parse_args()

    scrape_site(args.start_url, args.output)

if __name__ == "__main__":
    main()
EOF
```

## midifilter

```bash
echo '#!'"$(which python3)" >  midifilter
chmod +x midifilter
cat >> midifilter << 'EOF'
import requests
import sys

def check_content_type(url):
    try:
        response = requests.head(url, allow_redirects=True)
        content_type = response.headers.get('Content-Type', '')
        return 'application/octet-stream' in content_type
    except requests.RequestException:
        return False

for line in sys.stdin:
    url = line.strip()
    if check_content_type(url):
        print(url)
EOF
```

## imagefilter

```bash
echo '#!'"$(which python3)" >  imagefilter
chmod +x imagefilter
cat >> imagefilter << 'EOF'
import requests
import sys

def check_content_type(url):
    try:
        response = requests.head(url, allow_redirects=True)
        content_type = response.headers.get('Content-Type', '')
        return content_type.startswith('image/')
    except requests.RequestException:
        return False

for line in sys.stdin:
    url = line.strip()
    if check_content_type(url):
        print(url)
EOF
```

## pdffilter

```bash
echo '#!'"$(which python3)" >  pdffilter
chmod +x pdffilter
cat >> pdffilter << 'EOF'
# filter_pdf.py
import requests
import sys

def check_content_type(url):
    try:
        response = requests.head(url, allow_redirects=True)
        content_type = response.headers.get('Content-Type', '')
        return 'application/pdf' in content_type
    except requests.RequestException:
        return False

for line in sys.stdin:
    url = line.strip()
    if check_content_type(url):
        print(url)
EOF
```

# scrape_page

```
cd ~
cd ws
cd web_scraping
echo '#!'"$(which python3)" >  scrape_page
chmod +x scrape_page
cat >> scrape_page << 'EOF'
import requests
from bs4 import BeautifulSoup
from urllib.parse import urlparse, urljoin
import sys

def get_unique_urls(page_url):
    """Scrape the page for unique img and a tag URLs."""
    unique_urls = set()  # Use a set to store unique URLs
    try:
        response = requests.get(page_url)
        soup = BeautifulSoup(response.content, 'html.parser')

        # Extract URLs from <a> tags
        for link in soup.find_all('a', href=True):
            href = link['href']
            full_url = urljoin(page_url, href)
            unique_urls.add(full_url)

        # Extract URLs from <img> tags
        for img in soup.find_all('img', src=True):
            src = img['src']
            full_url = urljoin(page_url, src)
            unique_urls.add(full_url)

    except requests.RequestException as e:
        print(f"Error fetching page {page_url}: {e}", file=sys.stderr)

    return unique_urls

def main(url):
    """Main function to handle URL input and print unique URLs."""
    if not url:
        print("No URL provided.", file=sys.stderr)
        sys.exit(1)

    unique_urls = get_unique_urls(url)
    for url in unique_urls:
        print(url)

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python script.py <URL>", file=sys.stderr)
        sys.exit(1)

    main(sys.argv[1])
EOF
```

## download

```bash
cd ~
cd ws
cd web_scraping
echo '#!'"$(which python3)" >  download
chmod +x download
cat >> download << 'EOF'
import os
import sys
import requests
from urllib.parse import urlparse, unquote

def ensure_dir(directory):
    """Ensure that a directory exists; if not, create it."""
    if not os.path.exists(directory):
        os.makedirs(directory)

def download_file(url):
    """Download a file from a URL, preserving its path structure."""
    try:
        response = requests.get(url, stream=True)
        # Check if the request was successful (200 OK)
        if response.status_code == 200:
            parsed_url = urlparse(url)
            # Unquote URL to handle %20 spaces etc., and construct the local file path
            path = unquote(parsed_url.path)
            # Assuming all files are downloaded to a 'downloads' directory
            local_file_path = os.path.join('downloads', path.lstrip('/'))
            
            ensure_dir(os.path.dirname(local_file_path))
            
            with open(local_file_path, 'wb') as file:
                for chunk in response.iter_content(chunk_size=8192):
                    file.write(chunk)
            print(f"Downloaded: {url} -> {local_file_path}")
        else:
            print(f"Failed to download {url}: HTTP Status Code {response.status_code}", file=sys.stderr)
    except requests.RequestException as e:
        print(f"Error downloading {url}: {e}", file=sys.stderr)

def main():
    print("Enter URLs (one per line). Press Ctrl-D (Unix) or Ctrl-Z (Windows) followed by Enter when done:")
    for line in sys.stdin:
        url = line.strip()
        download_file(url)

if __name__ == "__main__":
    main()
EOF
```



