import re
from pronouncing import rhymes, phones_for_word
from rich.console import Console
from rich.text import Text
from collections import defaultdict

# Initialize Rich console for colored output
console = Console()

# Function to detect complex rhymes
def detect_complex_rhymes(word1, word2):
    # Get phonemes for each word
    phones1 = phones_for_word(word1)
    phones2 = phones_for_word(word2)
    
    # If either word has no phonemes, skip
    if not phones1 or not phones2:
        return False
    
    # Compare the last 2 phonemes for slant rhymes
    return phones1[0][-2:] == phones2[0][-2:]

# Function to assign colors to rhyme groups
def assign_colors(rhyme_groups):
    colors = ["red", "green", "yellow", "blue", "magenta", "cyan", "white"]
    color_map = {}
    for i, group in enumerate(rhyme_groups):
        color_map[group] = colors[i % len(colors)]
    return color_map

# Function to highlight rhymes in lyrics
def highlight_rhymes(lyrics):
    words = re.findall(r'\b\w+\b', lyrics)
    rhyme_groups = []
    rhyme_map = defaultdict(list)

    # Group words by their rhymes
    for i, word in enumerate(words):
        for j in range(i + 1, len(words)):
            if detect_complex_rhymes(word, words[j]):
                rhyme_map[word].append(words[j])

    # Create rhyme groups
    for word in words:
        if word in rhyme_map:
            rhyme_groups.append(tuple([word] + rhyme_map[word]))

    # Assign colors to rhyme groups
    color_map = assign_colors(rhyme_groups)

    # Highlight rhymes in the lyrics
    highlighted_text = Text()
    for line in lyrics.split('\n'):
        for word in re.findall(r'\b\w+\b|\W+', line):
            if word.strip() and any(word in group for group in rhyme_groups):
                for group in rhyme_groups:
                    if word in group:
                        highlighted_text.append(word, style=color_map[group])
                        break
            else:
                highlighted_text.append(word)
        highlighted_text.append('\n')

    return highlighted_text

# Main function
def main():
    # Example lyrics from Eminem's "Rap God"
    lyrics = """
    I'm beginning to feel like a Rap God, Rap God
    All my people from the front to the back nod, back nod
    Now who thinks their arms are long enough to slap box, slap box?
    They said I rap like a robot, so call me Rap-bot
    """

    # Highlight rhymes
    highlighted_lyrics = highlight_rhymes(lyrics)

    # Display highlighted lyrics
    console.print(highlighted_lyrics)

    # Option to save as text file
    save_option = input("Do you want to save the highlighted lyrics to a text file? (y/n): ")
    if save_option.lower() == 'y':
        with open("highlighted_lyrics.txt", "w") as file:
            file.write(highlighted_lyrics.plain)

if __name__ == "__main__":
    main()
