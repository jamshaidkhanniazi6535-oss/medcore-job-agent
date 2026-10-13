# medcore-job-agentimport requests
from bs4 import BeautifulSoup

# Your Telegram Username from CallMeBot
TELEGRAM_USER = "@Jamshaidniazi"

def search_global_healthcare_jobs():
    jobs = []
    # Target: Global healthcare job feed
    url = "https://remote.co/remote-jobs/healthcare/"
    headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"}
    
    try:
        response = requests.get(url, headers=headers)
        if response.status_code == 200:
            soup = BeautifulSoup(response.text, 'html.parser')
            listings = soup.find_all("div", class_="card-body")[:5]
            
            for item in listings:
                title_elem = item.find("a")
                if title_elem:
                    title = title_elem.text.strip()
                    link = title_elem.get("href", "")
                    
                    # Core domains: Scribe, Billing, Coding, VMA, Prior Auth
                    keywords = ["scribe", "billing", "coding", "virtual assistant", "prior auth", "healthcare", "medical"]
                    if any(kw in title.lower() for kw in keywords):
                        full_link = f"https://remote.co{link}" if link.startswith("/") else link
                        jobs.append(f"*{title}*\nApply: {full_link}")
    except Exception as e:
        print("Error fetching jobs:", e)
        
    return jobs

def send_telegram_alert(message):
    encoded_msg = requests.utils.quote(message)
    # CallMeBot Telegram Endpoint
    url = f"https://api.callmebot.com/text.php?user={TELEGRAM_USER}&text={encoded_msg}"
    response = requests.get(url)
    print("Telegram Alert Status:", response.status_code)

if __name__ == "__main__":
    found_jobs = search_global_healthcare_jobs()
    if found_jobs:
        digest = "MEDcore MENTORs Hourly Job Alert:\n\n" + "\n\n".join(found_jobs)
        send_telegram_alert(digest)
    else:
        print("No new matching global jobs found in this cycle.")
        
