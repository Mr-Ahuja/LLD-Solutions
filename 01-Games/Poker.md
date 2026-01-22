# Poker (Texas Hold'em) - Low Level Design

## Problem Statement

Design a Texas Hold'em Poker game with betting rounds, community cards, hand ranking, pot management, and multi-player support.

---

## Requirements

### Functional Requirements
1. Support 2-9 players
2. Betting rounds: Pre-flop, Flop, Turn, River
3. Player actions: Fold, Check, Call, Raise, All-in
4. Community cards (5 cards: 3 flop + 1 turn + 1 river)
5. Best 5-card hand evaluation
6. Pot management (main pot + side pots)
7. Blind rotation (small blind, big blind)
8. Winner determination and pot distribution

---

## Key Classes

```java
public class PokerGame {
    private List<Player> players;
    private Deck deck;
    private List<Card> communityCards;
    private Pot pot;
    private int dealerIndex;
    private int smallBlind;
    private int bigBlind;
    private GamePhase currentPhase;

    public void startHand() {
        deck.resetAndShuffle();
        communityCards.clear();
        pot.reset();

        // Post blinds
        postBlinds();

        // Deal hole cards (2 per player)
        dealHoleCards();

        // Betting rounds
        bettingRound();  // Pre-flop
        dealFlop();
        bettingRound();  // Flop
        dealTurn();
        bettingRound();  // Turn
        dealRiver();
        bettingRound();  // River

        // Showdown
        determineWinner();
    }

    private void postBlinds() {
        int sbIndex = (dealerIndex + 1) % players.size();
        int bbIndex = (dealerIndex + 2) % players.size();

        players.get(sbIndex).bet(smallBlind);
        players.get(bbIndex).bet(bigBlind);

        pot.addChips(smallBlind + bigBlind);
    }

    private void dealHoleCards() {
        for (Player player : players) {
            player.receiveCards(deck.dealMultiple(2));
        }
    }

    private void dealFlop() {
        deck.deal();  // Burn card
        communityCards.addAll(deck.dealMultiple(3));
        currentPhase = GamePhase.FLOP;
    }

    private void dealTurn() {
        deck.deal();  // Burn card
        communityCards.add(deck.deal());
        currentPhase = GamePhase.TURN;
    }

    private void dealRiver() {
        deck.deal();  // Burn card
        communityCards.add(deck.deal());
        currentPhase = GamePhase.RIVER;
    }

    private void bettingRound() {
        int currentBet = 0;
        int lastRaiseIndex = -1;
        int currentPlayerIndex = (dealerIndex + 3) % players.size();

        while (true) {
            Player player = players.get(currentPlayerIndex);

            if (player.hasFolded() || player.isAllIn()) {
                currentPlayerIndex = (currentPlayerIndex + 1) % players.size();
                continue;
            }

            PlayerAction action = player.decideAction(currentBet);

            switch (action.getType()) {
                case FOLD:
                    player.fold();
                    break;

                case CHECK:
                    if (currentBet > player.getCurrentBet()) {
                        throw new IllegalStateException("Cannot check");
                    }
                    break;

                case CALL:
                    int callAmount = currentBet - player.getCurrentBet();
                    player.bet(callAmount);
                    pot.addChips(callAmount);
                    break;

                case RAISE:
                    int raiseAmount = action.getAmount();
                    int totalBet = currentBet + raiseAmount;
                    int toBet = totalBet - player.getCurrentBet();
                    player.bet(toBet);
                    pot.addChips(toBet);
                    currentBet = totalBet;
                    lastRaiseIndex = currentPlayerIndex;
                    break;

                case ALL_IN:
                    int allInAmount = player.getChips();
                    player.bet(allInAmount);
                    pot.addChips(allInAmount);
                    break;
            }

            currentPlayerIndex = (currentPlayerIndex + 1) % players.size();

            // Betting round ends when we've circled back to last raiser
            if (currentPlayerIndex == lastRaiseIndex ||
                allPlayersActed(currentBet)) {
                break;
            }
        }
    }

    private boolean allPlayersActed(int currentBet) {
        int activePlayers = 0;
        for (Player player : players) {
            if (!player.hasFolded() && !player.isAllIn()) {
                if (player.getCurrentBet() != currentBet) {
                    return false;
                }
                activePlayers++;
            }
        }
        return activePlayers <= 1;
    }

    private void determineWinner() {
        List<Player> activePlayers = players.stream()
            .filter(p -> !p.hasFolded())
            .collect(Collectors.toList());

        if (activePlayers.size() == 1) {
            // Everyone else folded
            activePlayers.get(0).addChips(pot.getTotalChips());
            return;
        }

        // Evaluate hands
        Map<Player, HandRank> rankings = new HashMap<>();
        for (Player player : activePlayers) {
            List<Card> allCards = new ArrayList<>(player.getHand().getCards());
            allCards.addAll(communityCards);
            HandRank rank = evaluateBestHand(allCards);
            rankings.put(player, rank);
        }

        // Find winner(s)
        HandRank bestRank = rankings.values().stream()
            .max(Comparator.comparingInt(HandRank::getValue))
            .get();

        List<Player> winners = rankings.entrySet().stream()
            .filter(e -> e.getValue().equals(bestRank))
            .map(Map.Entry::getKey)
            .collect(Collectors.toList());

        // Split pot among winners
        int splitAmount = pot.getTotalChips() / winners.size();
        for (Player winner : winners) {
            winner.addChips(splitAmount);
        }
    }

    private HandRank evaluateBestHand(List<Card> allCards) {
        // Try all 5-card combinations from 7 cards
        // Return best hand
        // Implementation uses combinatorics to test all C(7,5) = 21 combinations
        return null;  // Simplified for brevity
    }
}

public enum GamePhase {
    PRE_FLOP,
    FLOP,
    TURN,
    RIVER,
    SHOWDOWN
}

public enum ActionType {
    FOLD,
    CHECK,
    CALL,
    RAISE,
    ALL_IN
}

public class PlayerAction {
    private ActionType type;
    private int amount;  // For RAISE

    public PlayerAction(ActionType type, int amount) {
        this.type = type;
        this.amount = amount;
    }

    public ActionType getType() { return type; }
    public int getAmount() { return amount; }
}

public class Pot {
    private int totalChips;
    private List<SidePot> sidePots;

    public void addChips(int chips) {
        totalChips += chips;
    }

    public int getTotalChips() {
        return totalChips;
    }

    public void reset() {
        totalChips = 0;
        sidePots.clear();
    }

    // Side pots for all-in scenarios
    public void createSidePot(int amount, List<Player> eligiblePlayers) {
        sidePots.add(new SidePot(amount, eligiblePlayers));
    }
}

public class SidePot {
    private int amount;
    private List<Player> eligiblePlayers;

    public SidePot(int amount, List<Player> eligiblePlayers) {
        this.amount = amount;
        this.eligiblePlayers = new ArrayList<>(eligiblePlayers);
    }

    public int getAmount() { return amount; }
    public List<Player> getEligiblePlayers() { return eligiblePlayers; }
}
```

---

## Design Decisions

### 1. **Betting Round State Machine**
**Decision**: Loop until all players have acted and bets are equal
**Reasoning**:
- Handles variable number of raises
- Accounts for folds and all-ins
- Natural termination when betting equalizes

### 2. **Community Cards List**
**Decision**: Store 5 community cards in single list
**Reasoning**:
- Simplifies hand evaluation (combine with hole cards)
- Easy to display board state
- Natural phased dealing (flop 3, turn 1, river 1)

### 3. **Side Pot Management**
**Decision**: Separate SidePot class for all-in scenarios
**Reasoning**:
- Handles complex all-in situations correctly
- Tracks which players eligible for each pot
- Prevents players from winning more than they bet

### 4. **Best Hand from 7 Cards**
**Decision**: Evaluate all C(7,5) = 21 combinations
**Reasoning**:
- Guaranteed to find best hand
- Only 21 combinations (fast)
- Simpler than heuristic-based approach

---

## Summary

Poker LLD demonstrates:
- **Complex State Management**: Multiple betting rounds, phases
- **Pot Management**: Main pot + side pots for all-ins
- **Hand Evaluation**: Best 5-card hand from 7 cards
- **Turn-Based Logic**: Dealer rotation, blind posting
- **Strategy Pattern**: Player actions (fold, call, raise)
