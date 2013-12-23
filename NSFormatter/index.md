---
layout: page
title: NSFormatter
---

Apple's documentation: http://developer.apple.com/documentation/Cocoa/Reference/Foundation/Classes/NSFormatter_Class/

----

**
On MacOsxDev, StevenFrank wrote:
**

I wrote a custom NSFormatter subclass that I want to apply to all the cells in a particular column of an NSTableView.  How do I do that?

If I was using the standard NSDateFormatter, for example, I'd just drag it off the palette and onto the column header.  That's all!

I tried instantiating my formatter subclass and dragging that onto the column header, but it wouldn't take.

Then I noticed that the table column had a "formatter" outlet, so I tried connecting that to the instance of my formatter, but it never seems to get called.  (In my subclass, I implemented the three methods that the NSFormatter docs say I must "at least" provide, and set breakpoints on them -- they never get hit.)

What'd I miss?

**
Stéphane Sudre replied:
**

When it comes to NSTableView and NSDataFormatter I don't use IB but I do that in the code with something like this:

    
 NSTableColumn *tColumn;
 id tPrototypeCell;
 
 tColumn=[IBarray_ tableColumnWithIdentifier:@"Date"];
 tPrototypeCell=[tColumn dataCell];
 [tPrototypeCell setFont:[NSFont labelFontOfSize:11.0]];
 [tPrototypeCell setFormatter:NSDateFormatter alloc] 
 initWithDateFormat:NSLocalizedStringFromTable(@"%m/%d/%y, %I:%M:%S %p",
 @"Firewall",@"Date formatter for the log") allowNaturalLanguage:NO;
 [tColumn setDataCell:tPrototypeCell];


Hope it helps.

----

The     /Developer/Examples/InterfaceBuilder/ProgressViewPalette demo contains code that sets up a custom Cell for use in IB, but the code is a little dense and hard to follow. What would be nice would be an IB palette that could take custom cells as plug-ins, so you could drop some simpler class in a folder for your custom cell and have it show up on the new palette. Hmmm.

-- AdamVandenberg

----

For a simple way to use a custom NSFormatter with IB, but without messing with IB palettes, try this:

http://www.cocoabuilder.com/archive/message/cocoa/2004/5/19/107487

----

I want to display a custom sheet when a formatter fails so I set myself as the delegate of the control and added the

    
 - (BOOL)control:(NSControl *)control didFailToFormatString:(NSString *)string errorDescription:(NSString *)error


method. The method gets called at the appropriate time and my sheet shows up. However, I also get an alert dialog that seems to be generated by the NSFormatter. It has two buttons "OK" and "Discard Changes". Is there anyway to prevent this alert dialog from showing?

----

What are you returning from that method? Docs say *Evaluate the error or query the user and return YES if string should be accepted as is, or NO if string should be rejected.* Maybe you should be using     - (void)control:(NSControl *)control didFailToValidatePartialString:(NSString *)string errorDescription:(NSString *)error instead?

----

No.     - (void)control:(NSControl *)control didFailToValidatePartialString:(NSString *)string errorDescription:(NSString *)error is called to validate partial strings, and never even gets called if partial string validation is off in the formatter.

----

I think (answer to first problem) you have to set a custom data cell (which can be an unmodified NSTextFieldCell, which is the default), then connect the *its* formatter outlet, not the table column's. Just drag the new cell onto your table column and click the little triangle on the right side of your column header. This selects the data cell in the Inspector. --JediKnil

----

After wrestling with NSFormatter subclassing for a day I've learned a few things  that may help others in the future that are stated in the documentation, but perhaps their implications are not self evident from the documentation.

An NSTextField can return a NIL object instead of an empty NSString when the user deletes the last character or the field is in fact empty. This can then be passed to the [NSFormatter stringForObjectValue]. The bad assumption is that you will always get an NSString from an NSTextField.  Not addressing this will cause apparent validation errors.  Not addressing the issues of unexpected classes, the following snippet might help get your intended results:

    
 - (NSString *)stringForObjectValue:(id)anObject
 {
 	NSString	*retVal;
 	
 	if ( anObject == nil ) {
 		retVal = [NSString stringWithString:@""];
 
 	} else {        // Should address [anObject class] == unknown as well
 		retVal = [NSString stringWithString:anObject];	
 	}
 	
 	return retVal;
 }


Although it is documented, narrative may help others. The isPartialStringValid methods are called after every key stroke, so these will be used to keep from allowing an invalid entry.  The getObjectValue method is also called after every key stroke. It isn't clear (or wasn't to me) that this is also how final validation occurs. It is legitimate (in fact expected) to keep passing back NIL results until you get exactly what you need.  So for instance if you are validating an email, even though it will pass the isPartialStringValid test as soon as you start entering text, the getObjectValue should pass NIL until you get a fully qualified email string.  The errorDescription pointer only becomes non-NIL when the user attempts to exit the field. It is also only used then, as is the getObjectValue method returning NIL or not-NIL.

The rest of the textShouldEndEditing and allowing the firstResponder to be released occurs automatically if the above is all done correctly.

It wasn't readily apparent (to me anyway) why the NSCell's isEntryAcceptable method wasn't needed anymore, but it is the interaction with the getObjectValue behaviour that has replaced it.

Hope this helps.

**
dcwood
**
