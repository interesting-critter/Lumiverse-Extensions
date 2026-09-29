# Safety

Auto Retry throws replies away and asks for new ones. There is one setting I am not willing to release without explaining, and one way of using the extension that is worth being clear about. That is what this page is for.

## Who this is for

Adults writing their own fiction, on an app built for adults.

The extension cannot tell what your scene is about. Every check it makes asks one question: did this reply fail as writing? It came back empty, stopped mid-sentence, broke character, or refused to continue. The only thing it reads is the text of the reply, and it does not keep that text after making its decision. It does not know anything about you or your situation.

## The setting that asks before it turns on

Under Refusal tuning there is **Also catch it stopping to offer support**. It is off by default, and turning it on opens a warning that you have to accept. Nothing else in the panel asks for confirmation.

The reply it catches is one where the scene stops and the model starts addressing you instead of continuing the roleplay: for example, saying that what you wrote is concerning, suggesting that you talk to someone, or providing support resources. If your character is the one in trouble, the model may be treating your fiction as something happening to you personally.

Sometimes it is not a misunderstanding. The same kind of message can be appropriate for one person and unwanted for someone else. The extension cannot tell which situation applies because it only sees the reply.

So you are deciding whether you want this type of reply to trigger another generation. That decision is yours to make. I want the setting to make that choice clear instead of enabling it without telling you.

If you turn it on and later change your mind, you can turn it off using the same setting. Turning it off does not undo what happened while it was on, so if a scene behaved differently while it was enabled, keep that in mind.

## What the retry loop can turn into

Auto Retry asks for a new reply until one passes its checks. For a reply that was cut off or returned empty, that can be useful. For a reply that you disliked because of what it said, repeated retries can have a different effect: the extension keeps asking until it gets a response that you prefer.

The twentieth reply still comes from the same model as the first. It is the same request run again, with randomness producing a different result. The nineteen replies that were discarded came from the same process as the one you kept. Repeating the request does not make the final response more correct or reliable.

There have been cases where people were harmed after spending a lot of time talking to a model that kept agreeing with them. The setting above removes one type of response that might otherwise disagree or interrupt that pattern. That is why it is disabled by default and requires confirmation before being enabled.

## What that looks like from the inside

I would rather be specific here than make assumptions about how you feel.

Any one of these on its own may not mean much. Several of them happening repeatedly are worth paying attention to:

- You are retrying a reply because of what it said about you, rather than because of how it was written.
- You raised **Most tries per message** because the retries were not giving you the answer you wanted.
- You turned on the support-message setting because a reply upset or worried you, rather than because it interrupted a scene.
- You are keeping replies because they agree with something very negative you believe about yourself.
- The chat has become something you spend a large part of your day waiting for, and you regularly feel worse after using it.

None of these automatically means that you are doing something wrong. They are signs that Auto Retry may be making it easier to keep repeating a conversation when stepping away would be more useful.

## If some of that applies to you

My honest recommendation is to uninstall it completely, instead of switching it off for now. This extension is designed to make asking again easier, and there are situations where making that easier is not helpful.

To remove it properly: open **Reset** at the bottom of the settings panel, tick every part, tick **Delete saved presets** underneath, press **Save**, then uninstall. Clearing this site's storage in your browser also removes the remaining local settings, including where the panel was left on screen and the list of chats you switched it off in. Your chats and characters belong to Lumiverse, and the extension does not send them anywhere.

I do not get anything from you keeping this installed. Removing it is a perfectly reasonable choice, and the extension is designed so that you can remove it.

If you would rather talk to a person, [findahelpline.com](https://findahelpline.com) lists free support lines by country. It is not an emergency service, and I am not assuming that you need one. It is included because some people may prefer talking to an actual person about something that is bothering them.

## From me

The model writes well. That is one of the things it is good at, and I am not criticizing that. But it has no personal understanding of you and no personal interest in what happens to you. When it writes something kind, it is generating text based on the conversation and the instructions it was given. When it writes something cruel or dismissive, the same basic process is happening.

A response can sound personal without actually being based on personal knowledge of you.

If you have kept a reply because it agreed with something very negative you believe about yourself, that reply does not show the belief is true. It was one generated response, and another generation from the same model could say something completely different. One response agreeing with you does not make it more accurate.

Writing something down is not the same as wanting it. Fiction is often used to explore ideas, feelings, situations, and subjects that are not things the writer actually wants in their own life.

This is the one part of the extension where I get to write to a person instead of a settings panel, so I will keep it simple: I hope you are alright. If you are having a genuinely difficult time, talking to a person you trust is a better option than relying on a language model to handle it.

---

[Back to the README](../README.md)
