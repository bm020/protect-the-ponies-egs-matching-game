# protect-the-ponies-egs-matching-game
Protect the Ponies – EGS Matching Game: A three-level, browser-based matching game about Equine Grass Sickness (EGS), built with Phaser 3 and designed for outreach and learning.

Protect the Ponies is a lightweight, three-level matching game that introduces Equine Grass Sickness (EGS) risk contexts, recognisable clinical signs, and simple protective actions in a child-friendly format. Built with Phaser 3 and deployed as a static site (GitHub Pages), it runs in any modern browser without plugins, uses no analytics, and includes clear educational messaging (not veterinary advice). Ideal for public engagement, school fairs, and quick demonstrations.







// Protect the Ponies – 3-Level Matching Game 
// Equine Grass Sicknes card memory game 

class Card extends Phaser.GameObjects.Container {
    constructor(scene, x, y, backTexture, frontTextureKey, logicalKey, width, height) {
        super(scene, x, y);
        this.frontTextureKey = frontTextureKey; // actual texture used
        this.logicalKey = logicalKey; // logical pair key
        this.backSprite = scene.add.sprite(0, 0, backTexture).setDisplaySize(width, height);
        this.frontSprite = scene.add.sprite(0, 0, this.frontTextureKey).setDisplaySize(width, height);
        this.add(this.backSprite);
        this.add(this.frontSprite);
        this.setSize(width, height);
        this.setInteractive({
            cursor: 'pointer'
        });

        this.isFlipped = false;
        this.frontSprite.setVisible(false);

        this.outline = scene.add.graphics();
        this.add(this.outline);
        this.outline.setVisible(false);

        this.originalX = x;
        this.originalY = y;

        this.on('pointerdown', () => {
            scene.flipCard(this);
        });

        this.on('pointerover', this.onHoverStart, this);
        this.on('pointerout', this.onHoverEnd, this);
    }

    flip() {
        this.scene.tweens.add({
            targets: this,
            scaleX: 0,
            duration: 150,
            onComplete: () => {
                this.isFlipped = !this.isFlipped;
                this.frontSprite.setVisible(this.isFlipped);
                this.backSprite.setVisible(!this.isFlipped);
                this.scene.tweens.add({
                    targets: this,
                    scaleX: 1,
                    duration: 150
                });
            }
        });
        this.scene.sound.play('flip');
    }

    showOutline(isMatch) {
        this.outline.clear();
        this.outline.lineStyle(4, isMatch ? 0x00ff00 : 0xff0000);
        this.outline.strokeRoundedRect(-this.width / 2, -this.height / 2, this.width, this.height, 12);
        this.outline.setVisible(true);
    }

    hideOutline() {
        this.outline.setVisible(false);
    }

    fadeOut() {
        this.scene.tweens.add({
            targets: this,
            alpha: 0,
            duration: 350,
            onComplete: () => {
                this.setVisible(false);
            }
        });
    }

    onHoverStart() {
        const pointer = this.scene.input.activePointer;
        const dx = this.x - pointer.x;
        const dy = this.y - pointer.y;
        const angle = Math.atan2(dy, dx);
        const distance = 10;

        this.scene.tweens.add({
            targets: this,
            x: this.originalX + Math.cos(angle) * distance,
            y: this.originalY + Math.sin(angle) * distance,
            duration: 100,
            ease: 'Cubic.easeOut'
        });
    }

    onHoverEnd() {
        this.scene.tweens.add({
            targets: this,
            x: this.originalX,
            y: this.originalY,
            duration: 100,
            ease: 'Cubic.easeOut'
        });
    }
}

class ProtectThePonies extends Phaser.Scene {
    constructor() {
        super();
        this.cards = [];
        this.flippedCards = [];
        this.canFlip = true;
        this.matchedCount = 0;
        this.gameInProgress = false;
        this.backgroundMusic = null;

        this.currentLevelIndex = 0; // 0,1,2
    }

    // ====== Content Maps ======
    get levelMap() {
        return {
            level1: [
                'cold_dry_weather',
                'disturbed_field',
                'no_known_cause',
                'overgrazed_pasture',
                'rotated_pasture',
                'young_horse'
            ],
            level2: [
                'trembling',
                'patchy_sweating',
                'dysphagia',
                'salivation',
                'ptosis',
                'lethargy'
            ],
            level3: [
                'call_vet',
                'collect_samples',
                'protein_analysis',
                'what_are_proteins',
                'stable_night',
                'protector'
            ]
        };
    }

    get egsFacts() {
        return {
            cold_dry_weather: 'EGS risk is higher in cold, dry spring weather.',
            overgrazed_pasture: 'Overgrazing may increase EGS risk.',
            disturbed_field: 'Recently disturbed fields are worth watching.',
            rotated_pasture: 'Rotating pastures helps protect ponies.',
            young_horse: 'Young adult horses are most affected.',
            no_known_cause: 'Scientists are still solving the EGS mystery.',
            trembling: 'Muscle tremors may be a sign of EGS.',
            patchy_sweating: 'Patchy sweating could mean your horse needs help.',
            dysphagia: 'Difficulty swallowing is a serious symptom.',
            salivation: 'Excess drooling can be linked to EGS.',
            ptosis: 'Droopy eyelids (ptosis) is a sign of muscle weakness in EGS.',
            lethargy: 'EGS can make horses feel very unwell.',
            call_vet: "Always call a vet if you're unsure.",
            collect_samples: 'Samples help scientists understand EGS.',
            protein_analysis: 'Protein analysis reveals clues in the lab.',
            what_are_proteins: 'Proteins help the body grow and function.',
            stable_night: 'Keep ponies safe in stables overnight.',
            protector: "You're doing a great job protecting ponies!"
        };
    }

    preload() {
        // Core art & audio
        this.load.image('card_front', 'https://play.rosebud.ai/assets/card_design_blank.png?aXvY'); // This can be kept as a fallback or removed if all cards have images
        this.load.image('card_back', 'https://play.rosebud.ai/assets/card_back.png?XKl5');

        this.load.audio('flip', 'https://play.rosebud.ai/assets/flipcard-91468.mp3?aUev');
        this.load.audio('match_success', 'https://play.rosebud.ai/assets/short-success-sound-glockenspiel-treasure-video-game-6346.mp3?GxzH');
        this.load.audio('background_music', 'https://play.rosebud.ai/assets/music-for-game-fun-kid-game-163649.mp3?H679');
        this.load.audio('carousel_music', 'https://play.rosebud.ai/assets/kids-cartoon-music-328596.mp3?ZtDQ');
        this.load.audio('win_sound', 'https://play.rosebud.ai/assets/level-win-6416.mp3?L9gj');

        this.load.image('gameCover', 'https://play.rosebud.ai/assets/Game_cover2.png?9qBk');
        this.load.image('winner', 'https://play.rosebud.ai/assets/winner.png?kEyi');
        // Carousel slides
        this.load.image('fact1', 'https://play.rosebud.ai/assets/fact1.png?e7IY');
        this.load.image('fact2', 'https://play.rosebud.ai/assets/fact2.png?G2f9');
        this.load.image('fact3', 'https://play.rosebud.ai/assets/fact3.png?vM5T');
        this.load.image('fact4', 'https://play.rosebud.ai/assets/fact4.png?IQMK');
        this.load.image('fact5', 'https://play.rosebud.ai/assets/fact5.png?N2sL');
        // Level 1 cards
        this.load.image('cold_dry_weather', 'https://play.rosebud.ai/assets/cold_dry_weather.png?oN6a');
        this.load.image('disturbed_field', 'https://play.rosebud.ai/assets/disturbed_field.png?fh6H');
        this.load.image('no_known_cause', 'https://play.rosebud.ai/assets/no_known_cause.png?HU3b');
        this.load.image('overgrazed_pasture', 'https://play.rosebud.ai/assets/overgrazed_pasture.png?dYSU');
        this.load.image('rotated_pasture', 'https://play.rosebud.ai/assets/rotated_pasture.png?9NEi');
        this.load.image('young_horse', 'https://play.rosebud.ai/assets/young_horse.png?pI1m');
        // Level 2 cards
        this.load.image('trembling', 'https://play.rosebud.ai/assets/trembling.png?JEiI');
        this.load.image('patchy_sweating', 'https://play.rosebud.ai/assets/patchy_sweating.png?dBQE');
        this.load.image('dysphagia', 'https://play.rosebud.ai/assets/dysphagia.png?JhxL');
        this.load.image('salivation', 'https://play.rosebud.ai/assets/salivation.png?Q3CR');
        this.load.image('ptosis', 'https://play.rosebud.ai/assets/ptosis.png?LnxS');
        this.load.image('lethargy', 'https://play.rosebud.ai/assets/lethargy.png?O4wL');
        // Level 3 cards
        this.load.image('call_vet', 'https://play.rosebud.ai/assets/call_vet.png?sVjB');
        this.load.image('collect_samples', 'https://play.rosebud.ai/assets/collect_samples.png?eD9c');
        this.load.image('protein_analysis', 'https://play.rosebud.ai/assets/protein_analysis.png?M1QU');
        this.load.image('what_are_proteins', 'https://play.rosebud.ai/assets/what_are_proteins.png?QMlD');
        this.load.image('stable_night', 'https://play.rosebud.ai/assets/stable_night.png?yXUc');
        this.load.image('protector', 'https://play.rosebud.ai/assets/protector.png?RNSM');

        // Helpful dev log for any load errors
        this.load.on('loaderror', (file) => {
            console.warn('[ASSET LOAD ERROR]', file && file.key, file && file.src);
        });
    }

    create() {
        this.gameInProgress = false;
        this.showCoverScreen();
    }

    // ====== UI Screens ======
    showCoverScreen() {
        this.cameras.main.setBackgroundColor('#d8f3dc');
        const {
            width,
            height
        } = this.sys.game.config;

        this.coverImage = this.add.image(width / 2, height / 2, 'gameCover');
        const scaleX = width / this.coverImage.width;
        const scaleY = height / this.coverImage.height;
        const scale = Math.min(scaleX, scaleY);
        this.coverImage.setScale(scale).setOrigin(0.5);

        this.startButton = this.add.text(width / 2, height * 0.85, 'Start Game', {
            font: '64px Arial',
            fill: '#0b3d2e',
            backgroundColor: '#ffffff',
            padding: {
                x: 30,
                y: 15
            },
            borderRadius: '15px'
        }).setOrigin(0.5).setInteractive();

        this.startButton.on('pointerdown', () => this.startGame(true));
    }

    startGame(newGame) {
        if (this.coverImage) {
            this.coverImage.destroy();
            this.startButton.destroy();
        }

        if (newGame) {
            this.currentLevelIndex = 0;
            if (this.cards) this.cards.forEach(card => card.destroy());
            this.cards = [];
            this.matchedCount = 0;
        }

        this.showLevelIntro(this.currentLevelIndex);
        this.gameInProgress = true;
    }

    setupGameForLevel(levelIndex) {
        this.cameras.main.setBackgroundColor('#d8f3dc');

        if (!this.backgroundMusic) {
            this.backgroundMusic = this.sound.add('background_music', {
                loop: true,
                volume: 0.5
            });
            this.backgroundMusic.play();
        } else if (!this.backgroundMusic.isPlaying) {
            this.backgroundMusic.play();
        }

        // Clear previous cards (if any)
        this.cards.forEach(c => c.destroy());
        this.cards = [];
        this.flippedCards = [];
        this.matchedCount = 0;
        this.canFlip = true;

        const levelKeys = [this.levelMap.level1, this.levelMap.level2, this.levelMap.level3][levelIndex];

        // Build pairs
        let cardData = [];
        levelKeys.forEach(key => {
            cardData.push({
                logicalKey: key,
                frontTex: key
            });
            cardData.push({
                logicalKey: key,
                frontTex: key
            });
        });

        Phaser.Utils.Array.Shuffle(cardData);

        const {
            width,
            height
        } = this.sys.game.config;
        const isPortrait = height > width;
        const cols = isPortrait ? 3 : 4;
        const rows = isPortrait ? 4 : 3;
        const totalCards = 12;
        const spacingX = width * 0.02; // 2% of width for horizontal spacing
        const spacingY = height * 0.02; // 2% of height for vertical spacing
        // Calculate card size based on available space
        const availableWidth = width - (spacingX * (cols + 1));
        const availableHeight = height - (spacingY * (rows + 1)) - 80; // reserve 80px for top/bottom margins
        const cardWidthBasedOnWidth = availableWidth / cols;
        const cardHeightBasedOnHeight = availableHeight / rows;
        const cardAspectRatio = 3 / 4; // width / height
        let cardWidth, cardHeight;
        if (cardWidthBasedOnWidth / cardHeightBasedOnHeight < cardAspectRatio) {
            cardWidth = cardWidthBasedOnWidth;
            cardHeight = cardWidth / cardAspectRatio;
        } else {
            cardHeight = cardHeightBasedOnHeight;
            cardWidth = cardHeight * cardAspectRatio;
        }
        const totalGridWidth = cols * cardWidth + (cols - 1) * spacingX;
        const totalGridHeight = rows * cardHeight + (rows - 1) * spacingY;
        const offsetX = (width - totalGridWidth) / 2 + cardWidth / 2;
        const offsetY = (height - totalGridHeight) / 2 + cardHeight / 2;
        cardData.forEach((data, i) => {
            const col = i % cols;
            const row = Math.floor(i / cols);
            const x = offsetX + col * (cardWidth + spacingX);
            const y = offsetY + row * (cardHeight + spacingY);
            const card = new Card(this, x, y, 'card_back', data.frontTex, data.logicalKey, cardWidth, cardHeight);
            this.add.existing(card);
            this.cards.push(card);
        });

        // this.showLevelBadge(levelIndex + 1);
    }

    showLevelBadge(n) {
        const {
            width
        } = this.sys.game.config;
        const badge = this.add.text(width / 2, 40, `Level ${n}`, {
            font: '40px Arial',
            color: '#0b3d2e',
            backgroundColor: '#a8e6cf',
            padding: {
                x: 16,
                y: 8
            }
        }).setOrigin(0.5, 0);
        this.tweens.add({
            targets: badge,
            alpha: 0,
            duration: 1200,
            delay: 800,
            onComplete: () => badge.destroy()
        });
    }

    // ====== Matching Logic ======
    flipCard(card) {
        if (!this.canFlip || card.isFlipped) return;

        card.flip();
        this.flippedCards.push(card);

        if (this.flippedCards.length === 2) {
            this.canFlip = false;
            this.checkMatch();
        }
    }

    checkMatch() {
        const [card1, card2] = this.flippedCards;
        const isMatch = card1.logicalKey === card2.logicalKey;

        card1.showOutline(isMatch);
        card2.showOutline(isMatch);

        if (isMatch) {
            this.sound.play('match_success');
            this.time.delayedCall(250, () => {
                this.showMatchBubble(card1.logicalKey, card1, card2);
            });
        } else {
            this.time.delayedCall(700, () => {
                card1.flip();
                card2.flip();
                card1.hideOutline();
                card2.hideOutline();
                this.resetFlippedCards();
            });
        }
    }

    resetFlippedCards() {
        this.flippedCards = [];
        this.canFlip = true;
    }

    // ====== Match Bubble (2 seconds above matched cards) ======
    showMatchBubble(logicalKey, card1, card2) {
        const cx = (card1.x + card2.x) / 2;
        const cy = Math.min(card1.y, card2.y) - 150;

        const container = this.add.container(cx, cy).setDepth(50);

        const text = this.add.text(0, 0, this.egsFacts[logicalKey] || '', {
            font: '24px Arial',
            color: '#000000',
            wordWrap: {
                width: 540
            },
            align: 'center'
        }).setOrigin(0.5);

        const padding = 16;
        const g = this.add.graphics();
        g.fillStyle(0xffffff, 1);
        g.lineStyle(3, 0x333333, 1);
        const bw = Math.min(580, text.width + padding * 2);
        const bh = text.height + padding * 2 + 12;
        g.fillRoundedRect(-bw / 2, -bh / 2, bw, bh, 16);
        g.strokeRoundedRect(-bw / 2, -bh / 2, bw, bh, 16);
        g.fillTriangle(-10, bh / 2 - 10, 10, bh / 2 - 10, 0, bh / 2 + 10);

        container.add(g);
        container.add(text);

        this.time.delayedCall(2000, () => {
            container.destroy();
            card1.hideOutline();
            card2.hideOutline();
            card1.fadeOut();
            card2.fadeOut();
            this.resetFlippedCards();

            this.matchedCount += 2;
            if (this.matchedCount === this.cards.length) {
                this.onLevelCompleted();
            }
        });
    }

    onLevelCompleted() {
        const levelMsgs = [
            'Level 1 Completed, Starting Level 2',
            'Level 2 Completed, Starting Level 3',
            'Level 3 Completed'
        ];

        const msg = levelMsgs[this.currentLevelIndex];
        const {
            width,
            height
        } = this.sys.game.config;

        const overlay = this.add.rectangle(width / 2, height / 2, width, height, 0x000000, 0.5).setDepth(60);
        const label = this.add.text(width / 2, height / 2, msg, {
            font: '56px Arial',
            color: '#ffffff',
            align: 'center'
        }).setOrigin(0.5).setDepth(61);

        this.time.delayedCall(1400, () => {
            overlay.destroy();
            label.destroy();

            if (this.currentLevelIndex < 2) {
                this.currentLevelIndex += 1;
                this.showLevelIntro(this.currentLevelIndex);
            } else {
                this.time.delayedCall(600, () => this.showWinnerPopup());
            }
        });
    }

    showWinnerPopup() {
        if (this.backgroundMusic && this.backgroundMusic.isPlaying) {
            this.backgroundMusic.stop();
        }
        this.sound.play('win_sound');

        const {
            width,
            height
        } = this.sys.game.config;
        this.cameras.main.setBackgroundColor('#a8e6cf');

        const winnerImage = this.add.image(width / 2, height / 2 - 60, 'winner');
        const scale = Math.min(width / winnerImage.width, (height * 0.7) / winnerImage.height);
        winnerImage.setScale(scale).setOrigin(0.5);

        const makeBtn = (y, label, onClick) => {
            const btn = this.add.text(width / 2, y, label, {
                font: '44px Arial',
                fill: '#ffffff',
                backgroundColor: '#a8e6cf',
                padding: {
                    x: 20,
                    y: 10
                }
            }).setOrigin(0.5).setInteractive({
                cursor: 'pointer'
            });
            btn.on('pointerdown', onClick);
            return btn;
        };

        const playAgainBtn = makeBtn(height * 0.78, 'Play Again', () => {
            this.scene.restart();
        });

        const learnMoreBtn = makeBtn(playAgainBtn.y + 70, 'Learn More About EGS', () => {
            this.scene.start('CarouselScene', {
                parentScene: this
            });
            this.scene.pause();
        });

        makeBtn(learnMoreBtn.y + 70, 'Visit Equine Grass Sickness Fund Website', () => {
            window.open('https://www.grasssickness.org.uk/', '_blank');
        });
    }
    showLevelIntro(levelIndex) {
        const levelTitles = [
            'LEVEL 1\nThe Farm Adventure 🐴',
            'LEVEL 2\nSpot the Sick Pony 😴',
            'LEVEL 3\nBe a Pony Protector! 💚'
        ];
        const levelSubtitles = [
            'Find the things that can make ponies poorly!',
            'Can you match the signs of an unwell pony?',
            'Learn how to help ponies stay safe and healthy!'
        ];
        const {
            width,
            height
        } = this.sys.game.config;
        this.cameras.main.setBackgroundColor('#d8f3dc');
        const titleText = levelTitles[levelIndex];
        const subtitleText = levelSubtitles[levelIndex];
        const fullText = `${titleText}\n${subtitleText}`;
        const introText = this.add.text(width / 2, height / 2, fullText, {
            font: '56px Arial',
            color: '#0b3d2e',
            align: 'center',
            wordWrap: {
                width: width * 0.9
            }
        }).setOrigin(0.5);
        this.time.delayedCall(4000, () => {
            introText.destroy();
            this.setupGameForLevel(levelIndex);
        });
    }
}

class CarouselScene extends Phaser.Scene {
    constructor() {
        super('CarouselScene');
        this.slides = [];
        this.currentIndex = 0;
    }

    init(data) {
        this.parentScene = data.parentScene;
    }

    create() {
        this.cameras.main.setBackgroundColor('#fffce7');
        this.music = this.sound.add('carousel_music', {
            loop: true,
            volume: 0.5
        });
        this.music.play();

        const {
            width,
            height
        } = this.sys.game.config;
        this.slides = ['fact1', 'fact2', 'fact3', 'fact4', 'fact5'].map(key => {
            const image = this.add.image(width / 2, height / 2, key).setVisible(false).setOrigin(0.5);
            const padding = 150;
            const scale = Math.min((width - padding) / image.width, (height - padding) / image.height);
            image.setScale(scale);
            return image;
        });

        this.createNavigation();
        this.showSlide(0);

        this.input.on('pointerdown', (pointer) => {
            this.startX = pointer.x;
        });
        this.input.on('pointerup', (pointer) => {
            const endX = pointer.x;
            if (this.startX - endX > 100) this.nextSlide();
            else if (endX - this.startX > 100) this.prevSlide();
        });
    }

    createNavigation() {
        this.backButton = this.add.text(100, this.sys.game.config.height - 70, 'Back', {
            font: '48px Arial',
            fill: '#000000'
        }).setInteractive();

        this.nextButton = this.add.text(this.sys.game.config.width - 100, this.sys.game.config.height - 70, 'Next', {
            font: '48px Arial',
            fill: '#000000'
        }).setOrigin(1, 0).setInteractive();

        this.backButton.on('pointerdown', this.prevSlide, this);
        this.nextButton.on('pointerdown', this.nextSlide, this);
    }

    showSlide(index) {
        const oldIndex = this.currentIndex;
        this.currentIndex = Phaser.Math.Clamp(index, 0, this.slides.length - 1);

        if (this.slides[oldIndex] && oldIndex !== this.currentIndex) {
            this.tweens.add({
                targets: this.slides[oldIndex],
                alpha: 0,
                duration: 800,
                onComplete: () => this.slides[oldIndex].setVisible(false)
            });
        } else if (this.slides[oldIndex]) {
            this.slides[oldIndex].setVisible(false);
        }

        this.slides[this.currentIndex].setAlpha(0).setVisible(true);
        this.tweens.add({
            targets: this.slides[this.currentIndex],
            alpha: 1,
            duration: 800
        });
        this.updateNavButtons();

        if (this.currentIndex === this.slides.length - 1) {
            this.showPlayAgainButton();
        } else if (this.playAgainButton) {
            this.playAgainButton.destroy();
        }
    }

    updateNavButtons() {
        this.backButton.setVisible(this.currentIndex > 0);
        this.nextButton.setVisible(this.currentIndex < this.slides.length - 1);
    }

    nextSlide() {
        if (this.currentIndex < this.slides.length - 1) this.showSlide(this.currentIndex + 1);
    }

    prevSlide() {
        if (this.currentIndex > 0) this.showSlide(this.currentIndex - 1);
    }

    showPlayAgainButton() {
        this.playAgainButton = this.add.text(this.sys.game.config.width / 2, this.sys.game.config.height - 70, 'Play Again', {
            font: '48px Arial',
            fill: '#ffffff',
            backgroundColor: '#a8e6cf',
            padding: {
                x: 20,
                y: 10
            }
        }).setOrigin(0.5).setInteractive();

        this.playAgainButton.on('pointerdown', () => {
            this.music.stop();
            this.scene.stop('CarouselScene');
            this.parentScene.scene.restart();
        });
    }
}

const config = {
    type: Phaser.AUTO,
    parent: 'renderDiv',
    width: 1920,
    height: 1080,
    scene: [ProtectThePonies, CarouselScene],
    antialias: true,
    roundPixels: false,
    scale: {
        mode: Phaser.Scale.FIT,
        autoCenter: Phaser.Scale.CENTER_BOTH
    }
};

window.phaserGame = new Phaser.Game(config);
