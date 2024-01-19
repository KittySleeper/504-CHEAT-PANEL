import flixel.FlxObject;
import flixel.text.FlxText;

var adminStrings:Dynamic = [
	[
		["Quit Game", "Closes Psych Engine."],
		["End Song", "Ends The Song."],
		["Botplay", "Botplay... Thats It."],
		["Speed Up", "Makes Song Faster."],
		["Slow Down", "Makes Song Slower."],
		["Stop Time", "Kinda Stops Time."],
		["Rgb Boyfriend", "Gay Bf. (kinda broke)"]
	],
	[
		["Break Notes", "Just Makes The Notes Weird."],
		["God Mode", "No Death."],
		["More Health", "Gives Some Health."],
		["Less Health", "Takes Away Some Health."],
		["Turn Off HP Colors", "I Guess You Can Debug?"],
		["Opponent Play", "Play As Dad."],
		["Placeholder", "Placeholder."]
	],
	[
		["Free Cam", "Control Camera With IJKL."],
		["Placeholder", "Placeholder."],
		["Placeholder", "Placeholder."],
		["Placeholder", "Placeholder."],
		["Placeholder", "Placeholder."],
		["Placeholder", "Placeholder."],
		["Placeholder", "Placeholder."]
	]
];

var version = "1.0";
var bg:FlxSprite;
var boardText:FlxText;
var buttonL:FlxSprite;
var buttonR:FlxSprite;
var buttons:Array<Array<Dynamic>> = [];
var texts:Array<Array<Dynamic>> = [];
var curButton = 0; // i have to do it like this okay :/
var curPage = 0;
var freeCam:FlxObject;

function onCreatePost() {
	FlxG.mouse.enabled = true;
	FlxG.mouse.visible = true;

	freeCam = new FlxObject(0, 0, 1, 1);
	freeCam.setPosition(game.camFollow.x, game.camFollow.y);

	regenMenu();
}

function onUpdatePost(elapsed:Float) {
	for (array in buttons) {
		if (FlxG.mouse.overlaps(array[1], camOther) && FlxG.mouse.justPressed) {
			if (array[0] != "Speed Up" && array[0] != "Slow Down" && array[0] != "Rgb Boyfriend" && array[0] != "More Health" && array[0] != "Less Health")
				array[1].antialiasing = !array[1].antialiasing;

			switch (array[0]) { // this is for when you only click things
				case "Quit Game":
					Sys.exit(1);

				case "End Song":
					game.health += 9999;
					game.finishSong(true);

				case "Botplay":
					game.cpuControlled = array[1].antialiasing;
					game.botplayTxt.visible = array[1].antialiasing;

				case "Speed Up":
					game.playbackRate += 0.1;

				case "Slow Down":
					game.playbackRate -= 0.1;

				case "Stop Time":
					if (array[1].antialiasing)
						game.playbackRate = 0;
					else
						game.playbackRate = 1;

				case "Rgb Boyfriend":
					rgbRecolor(game.boyfriend);

				case "Break Notes":
					for (note in game.unspawnNotes) {
						note.angle = FlxG.random.float(-180, 180);
						note.noteData = FlxG.random.int(0, 3);
					}

				case "Jack Notes":
					for (note in game.unspawnNotes) {
						note.isSustainNote = false;
						var colors:Array<String> = ['purple', 'blue', 'green', 'red'];
						note.animation.addByPrefix("newNote", colors[note.noteData] + "0");
						note.animation.play("newNote");
						note.alpha = 1;
						note.offsetX -= note.width / 2;
						note.scale.set(0.8, 0.8);
					}

				case "More Health":
					game.health += 0.2;

				case "Less Health":
					game.health -= 0.2;

				case "Opponent Play":
					for (note in game.unspawnNotes) {
						note.noAnimation = array[1].antialiasing;
						note.canBeHit = !note.canBeHit;
						note.mustPress = !note.mustPress;
					}

				case "Free Cam":
					if (array[1].antialiasing)
						FlxG.camera.follow(freeCam, "LOCKON", 0);
					else
						FlxG.camera.follow(camFollow, "LOCKON", 0);

					FlxG.camera.snapToTarget();
			}
		}

		if (array[1].antialiasing) {
			switch (array[0]) { // this is for things that will occur every second
				case "God Mode":
					game.health = 2;

				case "Rainbow Notes":
					for (note in game.unspawnNotes) {
						note.rgbShader.r += note.strumTime;
						note.rgbShader.g += note.strumTime;
						note.rgbShader.b += note.strumTime;
					}

				case "Turn Off HP Colors":
					game.healthBar.setColors(0xFFFF0000, 0xFF66FF33);

				case "Free Cam":
					if (FlxG.keys.pressed.I)
						freeCam.y -= 10;
					if (FlxG.keys.pressed.J)
						freeCam.x -= 10;
					if (FlxG.keys.pressed.L)
						freeCam.x += 10;
					if (FlxG.keys.pressed.K)
						freeCam.y += 10;
			}
		} else {
			switch (array[0]) { // this is for turning them off
				case "Turn Off HP Colors":
					game.reloadHealthBarColors();
			}
		}

		if (array[1].antialiasing)
			array[1].setColorTransform(1, 10, 1);
		else
			array[1].setColorTransform(0.5, 0.5, 0.5);
	}

	if (FlxG.mouse.overlaps(buttonL, camOther) && FlxG.mouse.justPressed && curPage != 0) {
		curPage--;
		regenMenu();
	}

	if (FlxG.mouse.overlaps(buttonR, camOther) && FlxG.mouse.justPressed && curPage != adminStrings.length - 1) {
		curPage++;
		regenMenu();
	}
}

function opponentNoteHitPost(daNote) {
	for (array in buttons) {
		if (array[1].antialiasing) {
			switch (array[0]) { // this is for thingies that occur when dad presses a note
				case "Opponent Play":
					var singAnims = ['singLEFT', 'singDOWN', 'singUP', 'singRIGHT'];

					game.boyfriend.playAnim(singAnims[daNote.noteData]);
			}
		}
	}
}

function goodNoteHitPost(daNote) {
	for (array in buttons) {
		if (array[1].antialiasing) {
			switch (array[0]) { // this is for thingies that occur when dad presses a note
				case "Opponent Play":
					var singAnims = ['singLEFT', 'singDOWN', 'singUP', 'singRIGHT'];

					game.dad.playAnim(singAnims[daNote.noteData]);
			}
		}
	}
}

function rgbRecolor(object:Dynamic) {
	var rgb = [FlxColor.RED, FlxColor.GREEN, FlxColor.BLUE];

	rgb.remove(object.color);

	FlxTween.color(object, 1.5, object.color, rgb[FlxG.random.int(0, rgb.length - 1)], {
		onComplete: function(tween) {
			rgbRecolor(object);
		}
	});
}

function regenMenu() {
	curButton = 0;

	for (killShit in [bg, boardText, buttonL, buttonR])
		if (killShit != null)
			killShit.kill();

	bg = new FlxSprite().makeGraphic(300, 400);
	bg.cameras = [game.camOther];
	add(bg);

	boardText = new FlxText(bg.x - 500, 10, FlxG.width, "504 CHEAT PANEL\nV:" + version.toUpperCase());
	boardText.setFormat(game.scoreTxt.font, 20, game.scoreTxt.color, game.scoreTxt.alignment, game.scoreTxt.borderStyle, game.scoreTxt.borderColor);
	boardText.cameras = [game.camOther];
	add(boardText);

	buttonL = new FlxSprite(0, 0).makeGraphic(15, 400, 0xFF303030);
	buttonL.cameras = [game.camOther];
	add(buttonL);

	buttonR = new FlxSprite(285, 0).makeGraphic(15, 400, 0xFF303030);
	buttonR.cameras = [game.camOther];
	add(buttonR);

	rgbRecolor(bg);

	for (button in buttons) {
		button[1].destroy();
		buttons.remove(button);
	}

	for (text in texts) {
		text[1].destroy();
		text[2].destroy();
		texts.remove(text);
	}

	texts = [];
	buttons = [];

	for (array in adminStrings[curPage]) {
		curButton++;

		var text = new FlxText(bg.x - 500, curButton * 50, FlxG.width, array[0]);
		var text2 = new FlxText(text.x, text.y + 17, FlxG.width, array[1]);

		var button = new FlxSprite(20, text.y).makeGraphic(250, 40, 0xFF303030);
		button.cameras = [game.camOther];
		button.antialiasing = array[2];
		add(button);

		text.setFormat(game.scoreTxt.font, 20, game.scoreTxt.color, game.scoreTxt.alignment, game.scoreTxt.borderStyle, game.scoreTxt.borderColor);
		text.cameras = [game.camOther];
		add(text);

		text2.setFormat(game.scoreTxt.font, 15, game.scoreTxt.color, game.scoreTxt.alignment, game.scoreTxt.borderStyle, game.scoreTxt.borderColor);
		text2.cameras = [game.camOther];
		add(text2);

		buttons.push([array[0], button]);
		texts.push([array[0], text, text2]);
	}
}
