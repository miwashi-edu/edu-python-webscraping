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

def scrape_site(start_url):
    """Scrape the site for local URLs."""
    base_url = f"{urlparse(start_url).scheme}://{urlparse(start_url).netloc}"
    visited_urls = set()
    urls_to_visit = [start_url]
    local_urls = set()

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
                        local_urls.add(absolute_url)
                        urls_to_visit.append(absolute_url)

            # Output the number of URLs found, updating the same line.
            sys.stdout.write(f"\rFound URLs: {len(local_urls)}" + " " * 20)
            sys.stdout.flush()

        except requests.RequestException as e:
            print(f"\nRequest failed: {e}", file=sys.stderr)

    # Ensure the final count is printed on a new line after the loop completes.
    print(f"\rTotal Found URLs: {len(local_urls)}".ljust(50))
    return local_urls

def write_urls_to_file(urls, file_path):
    """Write found URLs to a specified file."""
    with open(file_path, 'w') as file:
        for url in urls:
            file.write(f"{url}\n")
    print(f"\nURLs have been written to {file_path}")

def main():
    """Main function to parse arguments and control the script flow."""
    parser = argparse.ArgumentParser(description="Scrape a website for local URLs.")
    parser.add_argument("start_url", help="The starting URL to begin scraping from.")
    parser.add_argument("-o", "--output", help="Optional: Output file to write the URLs to.", default=None)

    args = parser.parse_args()

    found_urls = scrape_site(args.start_url)
    
    if args.output:
        write_urls_to_file(found_urls, args.output)
    else:
        # Print a newline after the final count to ensure it's easy to see.
        print("\nFinished scanning.")

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

