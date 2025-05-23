<script lang="ts">
    import ClipboardIcon from './clipboard_icon.svg';
    import { ClientUtils } from '$lib/client/ClientUtils';
    import { ServerAPI } from '$lib/client/ServerAPI';
    import jokerHat2Img from '$lib/components/images/hat_3.png?enhanced'
    import MainButton from '$lib/components/MainButton.svelte';
    import { CardsUtils } from '$lib/shared/CardsUtils';
    import PokerCardsDeck from '../../lib/components/poker_cards/PokerCardsDeck.svelte';
    import PasteFromClipboardButton from './PasteFromClipboardButton.svelte';
    import SecondaryButton from '$lib/components/SecondaryButton.svelte';
    const originalDeck = CardsUtils.standardDeck
    let deck = $state([...originalDeck])

    let encryptedDeck = $state('')
    let errorMessage = $state<string | undefined>(undefined)
    let decryptedMessage = $state<string | undefined>(undefined)

    let pageTitle = $state('Reveal message hidden in a deck of cards')
    let enableTypewriterCursor = $state(false)

    let currentOrderIsNotStandard = $state(false)
    let showDecryptButton = $derived(currentOrderIsNotStandard)

    async function onPasteFromClipboard() {
      navigator.clipboard.readText().then((clipboardText) => {
        deck = CardsUtils.normalizeDeck(clipboardText)
        decryptMessage()
      })
    }

    async function decryptMessage() {
      encryptedDeck = deck.join(' ')
      const validationResult = CardsUtils.validateDeck(encryptedDeck)
      if (!validationResult.success) {
        errorMessage = validationResult.reason
        return
      }

      const decoded = await ServerAPI.decode(encryptedDeck.split(' '))
      await writeTheMessage(decoded)
    }

    async function writeTheMessage(message: string) {
      enableTypewriterCursor = true
      await removeTitleWithTypewriterEffect()
      await ClientUtils.sleep(400)
      await setTitleWithTypewriterEffect('Your message is: ')
      await ClientUtils.sleep(400)
      await setTitleWithTypewriterEffect(`Your message id: ${message}`)
      enableTypewriterCursor = false
    }

    function removeTitleWithTypewriterEffect(): Promise<void> {
      const typewriterSpeed = 50

      return new Promise((resolve, reject) => {
        const removeLetter = () => {
          if (!pageTitle.length) {
            resolve()
            return
          }
          pageTitle = pageTitle.slice(0, pageTitle.length - 1)
          setTimeout(() => removeLetter(), typewriterSpeed)
        }

        removeLetter()
      })
    }

    function setTitleWithTypewriterEffect(newTitle: string): Promise<void> {
      const typewriterSpeed = 50

      return new Promise((resolve) => {
        const addLetter = () => {
          if (pageTitle.length === newTitle.length) {
            resolve()
            return
          }
          
          const nextChar = newTitle[pageTitle.length]
          pageTitle = `${pageTitle}${nextChar}`
          setTimeout(() => addLetter(), typewriterSpeed)
        }

        addLetter()
      })
    }
</script>

<style>
  @keyframes blink-text-cursor {
    from {
      border-right-color: black;
    }
    to {
      border-right-color: transparent;
    }
  }
  .title-with-cursor {
    border-right-width: 2px;
    animation: blink-text-cursor infinite .8s normal;
  }
</style>

<div class="mx-auto w-full pt-12 flex flex-col pb-12">
  <div class="self-center sm:w-52 motion-preset-seesaw motion-preset-stretch-sm motion-duration-2000">
    <enhanced:img src={jokerHat2Img} alt="Joker hat" />
  </div>
  <h2 class="text-3xl mb-8 text-primary-font-color dark:text-primary-font-color-dark font-mono self-center whitespace-pre-wrap h-[2.3rem]
             {enableTypewriterCursor ? 'title-with-cursor' : ''}">
    {pageTitle}
  </h2>
  <MainButton onClick={onPasteFromClipboard}>
    <img src={ClipboardIcon} class="w-6 h-6 mr-4" alt="Clipboard icon" />
    Paste from clipboard
  </MainButton>
  <h5 class="text-lg mt-8 dark:text-white font-mono self-center">... or click on the cards in the right order below:</h5>
  <PokerCardsDeck clickable={true} cards={deck} bind:currentOrderIsNotStandard={currentOrderIsNotStandard} />
  {#if true}
    <div class="mt-8 mx-auto">
      <SecondaryButton onClick={decryptMessage}>Decrypt</SecondaryButton>
    </div>
  {/if}
</div>
