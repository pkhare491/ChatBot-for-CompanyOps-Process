from difflib import SequenceMatcher
import textwrap

# Similarity check function
def similar(a, b):
    return SequenceMatcher(None, a, b).ratio()

# Knowledge base dictionary
knowledge_base = {
    "hi": "Hello! How can I help you today?",
    "business country": "Business Country is a vital data point used throughout Morningstar’s products and methodology. Business Country specifies the principal business location of the company.",
    "who is the process owner": "The process owner is Mithun Muni."
}

# Prefixes, suffixes, and abbreviations (all in lowercase)
prefixes = ["pt", "the", "a", "an", "the plc", "(the) plc","(The) PLC"]
suffixes = ["ltd", "limited", "inc", "corp", "company", "co", "plc", "ksc", "tbk", "(the) plc","a/s","saog"]
abbreviations = {
    "manufacturing": "mfg",
    "real estate investment trust": "REIT",
    "fundo de investimento imobiliario": "FII",
    "fundo de investimento imobiliario-": "FII",
    "electronics": "elect",
    "corporation": "corp",
    "company": "co",
    "materials": "mat",
    "limited": "ltd",
    "acquisition": "acq",
    "alternative": "alt",
    "advanced": "adv",
    "associated": "assoc",
    "building": "bldg",
    "business": "bus",
    "capital": "cap",
    "commercial": "comml",
    "communications": "comms",
    "consolidated": "cons",
    "development": "dev",
    "distribution": "dist",
    "engineering": "eng",
    "financial": "finl",
    "global": "glb",
    "group": "gr",
    "holdings": "hldgs",
    "industrial": "ind",
    "international": "intl",
    "management": "mgmt",
    "national": "ntl",
    "pharmaceuticals": "pharma",
    "private": "pvt",
    "resources": "res",
    "services": "servs",
    "solutions": "solns",
    "technology": "tech",
    "trust": "tr",
}

# List of tax haven countries
tax_haven_countries = [
    "aland islands", "anguilla", "bermuda", "bonaire, sint eustatius and saba", "bouvet island",
    "cayman islands", "channel islands", "christmas island", "cocos (keeling) islands", "curacao",
    "falkland islands", "faroe islands", "gibraltar", "guernsey", "heard island and mcdonald islands",
    "isle of man", "jersey", "marshall islands", "netherlands antilles", "norfolk island",
    "northern mariana islands", "south georgia and the south sandwich islands", "turks and caicos islands",
    "united states minor outlying islands", "virgin islands, british", "virgin islands, u.s.",
    "sint maarten", "monaco", "montserrat", "puerto rico", "saint barthelemy", "saint kitts and nevis",
    "saint martin", "st. lucia"
]

# Mapping of tax haven countries to their controlling countries
controlling_countries = {
    "aland islands": "finland", "anguilla": "uk", "bermuda": "uk", "bonaire, sint eustatius and saba": "netherlands",
    "bouvet island": "norway", "cayman islands": "uk", "channel islands": "united kingdom", "christmas island": "australia",
    "cocos (keeling) islands": "australia", "curacao": "netherlands", "falkland islands": "uk", "faroe islands": "denmark",
    "gibraltar": "uk", "guernsey": "uk", "heard island and mcdonald islands": "australia", "isle of man": "uk",
    "jersey": "uk", "marshall islands": "usa", "netherlands antilles": "netherlands", "norfolk island": "australia",
    "northern mariana islands": "usa", "south georgia and the south sandwich islands": "uk", "turks and caicos islands": "uk",
    "united states minor outlying islands": "usa", "virgin islands, british": "uk", "virgin islands, u.s.": "usa",
    "sint maarten": "netherlands", "monaco": "france", "montserrat": "uk", "puerto rico": "usa", "saint barthelemy": "france",
    "saint kitts and nevis": "uk", "saint martin": "france", "st. lucia": "united kingdom"
}

# Function to generate a short name
def generate_short_name(company_name):
    words = company_name.lower().split()  # Convert to lowercase and split

    # Remove one prefix
    for prefix in prefixes:
        prefix_words = prefix.split()
        if words[:len(prefix_words)] == prefix_words:
            words = words[len(prefix_words):]
            break

    # Remove one suffix
    for suffix in suffixes:
        suffix_words = suffix.split()
        if words[-len(suffix_words):] == suffix_words:
            words = words[:-len(suffix_words)]
            break

    # Replace words with abbreviations where applicable
    abbreviated_words = []
    i = 0
    while i < len(words):
        # Check for multi-word abbreviations
        found_abbreviation = False
        for phrase, abbreviation in abbreviations.items():
            phrase_words = phrase.split()
            if words[i:i+len(phrase_words)] == phrase_words:
                abbreviated_words.append(abbreviation)
                i += len(phrase_words)
                found_abbreviation = True
                break
        if not found_abbreviation:
            abbreviated_words.append(words[i])
            i += 1

    # Ensure the abbreviation "FII" is correctly placed at the end if it exists
    if "fii" in abbreviated_words:
        abbreviated_words.remove("fii")
        abbreviated_words.append("fii")

    # Join the abbreviated words
    abbreviated_name = " ".join(abbreviated_words)

    # Ensure the name does not end with '&'
    if abbreviated_name.endswith('&'):
        abbreviated_name = abbreviated_name[:-1].strip()


    # Shorten the name if it exceeds 25 characters
    if len(abbreviated_name) > 25:
        abbreviated_name = textwrap.shorten(abbreviated_name, width=25, placeholder="...").strip()

    # Ensure the shortened name does not end with '&'
    if abbreviated_name.endswith('&'):
        abbreviated_name = abbreviated_name[:-1].strip()

    return abbreviated_name.title()

# Function to determine the business country
def ask_business_country_questions():
    domicile = input("What is the domicile country? ").strip().lower()
    listing = input("What is the listing country? ").strip().lower()
    hq = input("What is the headquarters country? ").strip().lower()

    # Check consistency among the three factors
    if domicile == listing == hq:
        business_country = domicile
    else:
        geographical_revenue = input("Is there a geographical revenue country? (Yes/No): ").strip().lower()
        if geographical_revenue == 'yes':
            revenue_country = input("What is the geographical revenue country? ").strip().lower()
            business_country = revenue_country
        else:
            business_country = hq

    # Check if the determined business country is a tax haven
    if business_country in tax_haven_countries:
        if hq not in tax_haven_countries:
            business_country = hq
        elif listing not in tax_haven_countries:
            business_country = listing
        elif domicile not in tax_haven_countries:
            business_country = domicile
        else:
            # All three factors are tax havens, use the controlling country
            business_country = controlling_countries.get(business_country, business_country)

    print(f"Your business country is {business_country.capitalize()}.")

# Main chatbot loop
def chatbot():
    print("🤖 Hello! I am your process assistant chatbot. Ask me anything about the process!")

    while True:
        user_question = input("You: ").strip().lower()

        if user_question in ["exit", "quit", "bye"]:
            print("🤖 Goodbye! Feel free to ask me anytime. 😊")
            break

        # Check if the user is asking for a short name
        if "short name for" in user_question:
            company_name = user_question.split("short name for", 1)[1].strip()
            short_name = generate_short_name(company_name)
            print(f"🤖 The short name for '{company_name}' is '{short_name}'.")
            continue

        # Check if the user is asking about the business country
        if "business country" in user_question:
            print(f"🤖 {knowledge_base['business country']}")
            ask_business_country_questions()
            continue

        # Search in the knowledge base
        best_match = None
        highest_similarity = 0

        for question, answer in knowledge_base.items():
            similarity = similar(user_question, question)
            if similarity > highest_similarity and similarity >= 0.7:  # Threshold is 0.7
                highest_similarity = similarity
                best_match = answer

        if best_match:
            print(f"🤖 {best_match}")
        else:
            print("🤖 I'm sorry, I didn't quite understand that. Could you please rephrase?")

# Start the chatbot
chatbot()
