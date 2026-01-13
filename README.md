radio.on()
radio.setGroup(7) // Both Micro:bits must use same group

let prankMode = false
let buttonSequence: string[] = []
let sequenceTimer = 0
let specialMessage = {name: "SECRET", melody: "C5 B A G F E D C"}

// -------------------- SHOW CREATOR --------------------
basic.showString("Ahmed Jaballah") // Scroll name at startup

// Define 10 pre-recorded voice messages
let messages = [
    {name: "HELLO", melody: "C D E F G A B C5"},
    {name: "YES", melody: "C5 B A G F E D C"},
    {name: "NO", melody: "C C G G A A G -"},
    {name: "HI", melody: "E F G A B C5 D5 E5"},
    {name: "OK", melody: "G F E D C B A G"},
    {name: "WOW", melody: "C D E D C D E D"},
    {name: "BYE", melody: "C5 D5 E5 F5 G5 A5 B5 C6"},
    {name: "HELP", melody: "G A B C D E F G"},
    {name: "FUN", melody: "E D C D E D C D"},
    {name: "YAY", melody: "C C G G A A G -"}
]

// -------------------- BUTTONS --------------------
// Button A → Send message 1 (HELLO)
input.onButtonPressed(Button.A, function () {
    sendMessage(messages[0])
    registerSequence("A")
})

// Button B → Send message 2 (YES)
input.onButtonPressed(Button.B, function () {
    sendMessage(messages[1])
    registerSequence("B")
})

// Button A+B → Send message 3 (NO)
input.onButtonPressed(Button.AB, function () {
    sendMessage(messages[2])
    registerSequence("AB")
})

// Long press combos can send extra messages
input.onButtonPressed(Button.A, function () {
    sendMessage(messages[3]) // HI
})

input.onButtonPressed(Button.B, function () {
    sendMessage(messages[4]) // OK
})

// -------------------- FUNCTIONS --------------------

// Send message via radio and play locally
function sendMessage(msg: {name: string, melody: string}) {
    radio.sendString(msg.name)
    playMessage(msg)
}

// Play melody + LED effects + scroll text
function playMessage(msg: {name: string, melody: string}) {
    // LED siren effect
    for (let i = 0; i < 3; i++) {
        led.plotAll()
        basic.pause(100)
        basic.clearScreen()
        basic.pause(100)
    }
    // Play melody
    music.playMelody(msg.melody, 120)
    // Scroll message name
    basic.showString(msg.name)
    // Optional: Scroll creator name after each message
    basic.showString("Ahmed Jaballah")
}

// Register button sequence for special message
function registerSequence(btn: string) {
    buttonSequence.push(btn)
    sequenceTimer = input.runningTime()
}

// Check if special sequence is complete
basic.forever(function () {
    // Clear sequence if > 3 seconds
    if (buttonSequence.length > 0 && input.runningTime() - sequenceTimer > 3000) {
        buttonSequence = []
    }

    // Detect special sequence: A → B → AB → triggers SECRET
    if (buttonSequence.join(",") == "A,B,AB") {
        sendMessage(specialMessage)
        buttonSequence = []
    }
})

// -------------------- RADIO RECEIVE --------------------
radio.onReceivedString(function (receivedString) {
    for (let i = 0; i < messages.length; i++) {
        if (messages[i].name == receivedString) {
            playMessage(messages[i])
        }
    }
    if (receivedString == specialMessage.name) {
        playMessage(specialMessage)
    }
})

// -------------------- PRANK MODE --------------------
input.onGesture(Gesture.Shake, function () {
    if (prankMode) {
        let randIndex = Math.randomRange(0, messages.length - 1)
        playMessage(messages[randIndex])
    }
})

// -------------------- IDLE LED --------------------
basic.forever(function () {
    led.plot(2, 2)
    basic.pause(200)
    basic.clearScreen()
    basic.pause(200)
})
