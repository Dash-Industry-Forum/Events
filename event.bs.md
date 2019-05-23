<!-- Document metadata follows. The below sections are used by the document compiler and are not directly visible. -->

<pre class="metadata">
Revision: 1.0

Title: DASH player's Application Events and Timed Metadata Processing Models and APIs
Status: LD
Shortname: application events
URL: https://dashif.org/guidelines/name-of-doc
Issue Tracking: GitHub https://github.com/Dash-Industry-Forum/Events/issues
Repository: https://github.com/Dash-Industry-Forum/Event GitHub
Editor: DASH Industry Forum

Default Highlight: text
<!-- Enabling line numbers breaks code blocks in PDF! (2018-10-02) -->
Line Numbers: off
Markup Shorthands: markdown yes
Boilerplate: copyright off, abstract off
Abstract: None
</pre>

#External Defintion - (to be removed)

<dfn element>SegmentBase</dfn>
<dl dfn-type="element-attr" dfn-for="SegmentBase">
<dfn>PresentationTimeOffset</dfn> 
<dfn>timescale</dfn>
</dl>
<dfn>Presentation time offset</dfn>  
<dfn>time scale</dfn>
<dfn>cmaf</dfn> 

# DASH player architecture for processing DASH events and timed metadata tracks # {#event-architecture}

Figure 1 demonstrates a generic architecture of the DASH player including DASH Events and timed metadata tracks processing models.

<figure>
	<img src="Images/eventclientarch.png" />
    <figcaption>Figure 1: DASH player architecture including the inband Event and Application-related timed metadata handling</figcaption>
</figure>

In the above figure:

1. DASH player processes the MPD. If the manifest
    includes any MPD Events, it parses them and appends them to the Event & Timed Metadata Buffer.

2. Based on the MPD, DASH player manages fetching and parsing the Segments before appending them into the media decoder input buffer (named 'Media Buffer' in the Figure 1).

3. Parsing a Segment includes:
    
    1. parse the high-level boxes such as Segment Index (sidx) and Event Message boxes, and append Event Message boxes to the Event & Metadata Buffer.
    2. For an Application-related timed metadata track, extracting the data samples, and appending them to the Event & Metadata Buffer.
    3. For media segments, parse the segments and append them to Media Buffer.

4.  The DASH player-specific Events are passed to the DASH player control function (named 'DASH Client Control, Selection & Heuristic Logic' in Figure 1), while the Application-related Events and timed metadata track samples are passed to the Event & Metadata Synchronizer and Dispatcher function.

5. If an Application is subscribed to a specific Event or timed metadata stream, dispatch the corresponding event instances or timed metadata samples, according to the dispatch mode:
    1. For [=On-receive=] dispatch mode, dispatch the Event information or timed metadata samples as soon as they are received(or no later than <var>AT</var>).
    2. For [=On-start=] dispatch mode, dispatch the Event information or timed metadata samples at their associated presentation time, using the synchronization signal from the media decoder.

# Event and Timed metadata sample timing models # {#event-metadata-timing}

## Inband Event timing parameters ## {#Inband-event-timing}

Figure 2 presents the timing of an inband Event along the media timeline:
<figure>
	<img src="Images/inbandeventtiming.png" />
    <figcaption>Figure 2: The inband event timing parameter on the media timeline</figcaption>
</figure>

As shown in Figure 2, every inband Event can be described with three timing parameters on the media timeline:

1. Event Arrival Time (<var>AT</var>) which is the earliest presentation time of the Segment containing the Event Message box.

2. Event Presentation/Start Time (<var>ST</var>) which is the moment in the media timeline that the Event becomes active.

3. Event duration (<var>DU</var>): the duration for which the Event is active

An inband Event is inserted in the beginning of a Segment. Since each media segment has an earliest presentation time equal to (<var>AT</var>), <var>AT</var> of the Segment carrying the Event Message box can be considered as the location of that box on the media timeline. The DASH player has to fetch and parse the Segment before or at its <var>AT</var> (at <var>AT</var> when it's assumed that the decoding and rendering of the segment incurs practically zero delay). Therefore, the Event inserted in a Segment at its <var>AT</var> time will be ready to be processed and fetched no later than <var>AT</var> on the media timeline.

The second timing parameter is Event Presentation/Start Time (<var>ST</var> ). <var>ST</var> is the moment in the media timeline that the Event becomes active. This value can be calculated using the parameters included in Event Message box.

The third parameter is Event Duration (<var>DU</var> ), the duration for which the Event is considered to be active. <var>DU</var>  is also signaled in the Event Message box using a specific value.

## Event message box format and event timing parameters ## {#emsg-format}

Table 1 shows the emsg box format in DASH:

<figure>
<table class=MsoTableGrid border=1 cellspacing=0 cellpadding=0 bgcolor="#DDDDDD"
 style='border-collapse:collapse;border:none'>
 <tr>
  <td width=623 valign=top style='width:600pt;border:solid #4472C4 2.25pt;
  padding:0in 5.4pt 0in 5.4pt'>
<table>
<tr>
 <td colspan="4">aligned(8) class DASHEventMessageBox extends FullBox (‘emsg’, version, flags = 0){</td>
</tr>
<tr><tr  class="odd">
<td></td>
<td colspan="2">if (version==0) {</td>
<td></td>
</tr></tr>

<tr class="even">
<td></td>
<td></td>
<td>string</td>
<td><dfn>scheme_id_uri</dfn>;</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td>string</td>
<td><dfn>value</dfn>;</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>unsigned int(32)</td>
<td><dfn>timescale</dfn>;</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td>unsigned int(32)</td>
<td><dfn>presentation_time_delta</dfn>;</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>unsigned int(32)</td>
<td><dfn>event_duration</dfn>;</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td>unsigned int(32)</td>
<td><dfn>id</dfn>;</td>
</tr>
<tr class="even">
<td></td>
<td colspan="2">} else if (version==1) {</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td>unsigned int(32)</td>
<td>[=timescale=];</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>unsigned int(64)</td>
<td><dfn>presentation_time</dfn>;</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td>unsigned int(32)</td>
<td>[=event_duration=];</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>unsigned int(32)</td>
<td>[=id=];</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td>string</td>
<td>[=scheme_id_uri=];</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>string</td>
<td>[=value=];</td>
</tr>
<tr class="odd">
<td></td>
<td colspan="2">}</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td colspan="2">unsigned int(8)</td>
<td><dfn>message_data()</dfn>;</td>
</tr>
<tr class="odd">
<td colspan="4">}</td>
</tr>
</tbody>
</table>
 <p class=MsoNormal></p>
  </td>
 </tr>
</table>
<figcaption>Table 1: The emsg box format and parameters</figcaption>
</figure>



The <var>ST</var> of an event can be calculated using values in its emsg box:


<figure>

$$ST = \begin{cases}
AT + \frac{presentation\_time\_delta}{timescale} \space
\qquad\qquad\qquad\qquad\qquad\qquad\qquad\qquad\qquad\qquad  version=0\\ 
PeriodStart +  \frac{<{SegmentBase/presentationTimeOffset}>}{<{SegmentBase/timescale}>} + \frac{presentation\_time}{timescale}\qquad \qquad version=1
\end{cases}
$$

<figcaption>Equation 1: Event Start Time of an inband event</figcaption>
</figure>


Where <var>PeriodStart</var> is the corresponding Period‘s start time, and <{SegmentBase/presentationTimeoffset}> and <{SegmentBase/timescale}> are the [=Presentation Time Offset=] (PTO) and [=time scale=] of the corresponding Represenation.

Note: <var>ST</var> is always equal to or larger than <var>AT</var> in both versions of emsg.

Note: Since the media sample timescales might be different than emsg's timescale, <var>ST</var> might not line up with a media sample if different timescales are used.

Note: If various Adaptation Sets carry the same events, different
Adaptation Sets/Representations with different PTOs, the
[=presentation_time_delta=] and/or [=presentation_time=] values might be different per Adaptation Set/Representation, i.e. the same emsg box can not be replicated over multiple Representations and/or Adaptations Sets.

Note: In the case of [=CMAF=], <var>PeriodStart</var> is the CMAF track's earliest presentation time.

In this document, we use the following common variable names instead of some of above variables to harmonize parameters between Inband events, MPD events, and timed metadata samples:

- <var>scheme_id</var> = [=scheme_id_uri=]
- <var>duration</var> = [=event_duration=]
- <var>message_data</var> = [=message_data()=]

## MPD Events timing model ## {#mpd-event-timing}

MPD Events carry the similar data model as inband Events. However, the former type is are carried in the MPD, under the Period elements. Each Period event has <{EventStream}> element(s), defining the <{EventStream/schemeIdUri}>, <{EventStream/value}> , <{EventStream/timescale}> and a sequences of <{Event}> elements. Each event may have <{Event/presentationTime}>, <{Event/duration}>, <{Event/id}> and <{Event/messageData}> attributes, as shown in Table 2.


<figure>
<table class=MsoTableGrid border=1 cellspacing=0 cellpadding=0
 style='border-collapse:collapse;border:none'>
 <tr>
  <td width=623 valign=top style='width:800pt;border:solid #4472C4 2.25pt;
  padding:0in 5.4pt 0in 5.4pt'>
  <table class=MsoNormalTable border=1 cellspacing=0 cellpadding=0 width="99%"
   style='width:99.2%;border-collapse:collapse;border:none'>
   <tr>
    <td width="29%" colspan=3 valign=top style='width:29.84%;border:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=TableCell style='margin-top:3.0pt;page-break-after:avoid'><b>Element
    or Attribute Name</b></p>
    </td>
    <td width="13%" valign=top style='width:13.84%;border:solid black 1.0pt;
    border-left:none;padding:0in 5.4pt 0in 5.4pt'>
    <p class=TableCell style='margin-top:3.0pt;page-break-after:avoid'><b>Use</b></p>
    </td>
    <td width="56%" valign=top style='width:56.32%;border:solid black 1.0pt;
    border-left:none;padding:0in 5.4pt 0in 5.4pt'>
    <p class=TableCell style='margin-top:3.0pt;page-break-after:avoid'><b>Description</b></p>
    </td>
   </tr>
   <tr>
    <td width="2%" valign=top style='width:2.52%;border-top:none;border-left:
    solid black 1.0pt;border-bottom:solid black 1.0pt;border-right:none;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><b><span style='font-size:9.0pt'>&nbsp;</span></b></p>
    </td>
    <td width="27%" colspan=2 valign=top style='width:27.32%;border-top:none;
    border-left:none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><b><span style='font-size:9.0pt;font-family:"Courier New"'><dfn element>EventStream</dfn></span></b></p>
    </td>
    <td width="13%" valign=top style='width:13.84%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=center style='text-align:center'><b><span
    style='font-size:9.0pt'>&nbsp;</span></b></p>
    </td>
    <td width="56%" valign=top style='width:56.32%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:9.0pt'>specifies event
    Stream</span></p>
    </td>
   </tr>
   <tr>
    <td width="2%" valign=top style='width:2.52%;border-top:none;border-left:
    solid black 1.0pt;border-bottom:solid black 1.0pt;border-right:none;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><b><span style='font-size:9.0pt'>&nbsp;</span></b></p>
    </td>
    <td width="2%" valign=top style='width:2.64%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><b><span style='font-size:9.0pt'>&nbsp;</span></b></p>
    </td>
    <td width="24%" valign=top style='width:24.68%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal style='margin-top:3.0pt;page-break-after:avoid'><span
    style='font-size:9.0pt;font-family:"Courier New"'>@xlink:href</span></p>
    </td>
    <td width="13%" valign=top style='width:13.84%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=center style='margin-top:3.0pt;text-align:center;
    page-break-after:avoid'><span style='font-size:9.0pt'>O</span></p>
    </td>
    <td width="56%" valign=top style='width:56.32%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:9.0pt'>specifies </span><span
    style='font-size:9.0pt'>a reference to an external </span><b><span
    style='font-size:9.0pt;font-family:"Courier New"'>EventStream</span></b><span
    style='font-size:9.0pt'> element</span></p>
    </td>
   </tr>
   <tr>
    <td width="2%" valign=top style='width:2.52%;border-top:none;border-left:
    solid black 1.0pt;border-bottom:solid black 1.0pt;border-right:none;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.64%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="24%" valign=top style='width:24.68%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal style='margin-top:3.0pt;page-break-after:avoid'><span
    style='font-size:9.0pt;font-family:"Courier New"'>@xlink:actuate</span></p>
    </td>
    <td width="13%" valign=top style='width:13.84%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=center style='text-align:center'><span
    style='font-size:9.0pt'>OD</span></p>
    <p class=MsoNormal align=center style='margin-top:3.0pt;text-align:center;
    page-break-after:avoid'><span style='font-size:9.0pt'>default:<br>
    </span><span style='font-size:9.0pt;font-family:"Courier New"'>onRequest</span></p>
    </td>
    <td width="56%" valign=top style='width:56.32%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=left style='margin-top:3.0pt;margin-right:0in;
    margin-bottom:10.0pt;margin-left:0in;text-align:left;page-break-after:avoid'><span
    style='font-size:9.0pt'>specifies </span><span style='font-size:9.0pt'>the
    processing instructions, which can be either &quot;</span><span
    style='font-size:9.0pt;font-family:"Courier New"'>onLoad</span><span
    style='font-size:9.0pt'>&quot; or &quot;</span><span style='font-size:9.0pt;
    font-family:"Courier New"'>onRequest</span><span style='font-size:9.0pt'>&quot;.</span></p>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:9.0pt'>This attribute shall
    not be present if the </span><span style='font-size:9.0pt;font-family:"Courier New"'>@xlink:href</span><span
    style='font-size:9.0pt'> attribute is not present.</span></p>
    </td>
   </tr>
   <tr>
    <td width="2%" valign=top style='width:2.52%;border-top:none;border-left:
    solid black 1.0pt;border-bottom:solid black 1.0pt;border-right:none;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.64%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="24%" valign=top style='width:24.68%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal style='margin-top:3.0pt;page-break-after:avoid'><span
    style='font-size:9.0pt;font-family:"Courier New"'><dl dfn-type="element-attr" dfn-for="EventStream">@<dfn>schemeIdUri</dfn></dl></span></p>
    </td>
    <td width="13%" valign=top style='width:13.84%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=center style='margin-top:3.0pt;text-align:center;
    page-break-after:avoid'><span style='font-size:9.0pt'>M</span></p>
    </td>
    <td width="56%" valign=top style='width:56.32%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:9.0pt'>identifies the
    message scheme. The string may use URN or URL syntax.&nbsp; When a URL is
    used, it is recommended to also contain a month-date in the form mmyyyy;
    the assignment of the URL must have been authorized by the owner of the
    domain name in that URL on or very close to that date.&nbsp; A URL may
    resolve to an Internet location, and a location that does resolve may store
    a specification of the message scheme.</span></p>
    </td>
   </tr>
   <tr>
    <td width="2%" valign=top style='width:2.52%;border-top:none;border-left:
    solid black 1.0pt;border-bottom:solid black 1.0pt;border-right:none;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.64%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><b><span style='font-size:9.0pt'>&nbsp;</span></b></p>
    </td>
    <td width="24%" valign=top style='width:24.68%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal style='margin-top:3.0pt;page-break-after:avoid'><span
    style='font-size:9.0pt;font-family:"Courier New"'><dl dfn-type="element-attr" dfn-for="EventStream">@<dfn>value</dfn></dl></span></p>
    </td>
    <td width="13%" valign=top style='width:13.84%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=center style='margin-top:3.0pt;text-align:center;
    page-break-after:avoid'><span style='font-size:9.0pt'>O</span></p>
    </td>
    <td width="56%" valign=top style='width:56.32%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:9.0pt'>specifies the value
    for the event stream element. The value space and semantics must be defined
    by the owners of the scheme identified in the </span><span
    style='font-size:9.0pt;font-family:"Courier New"'>@schemeIdUri</span><span
    style='font-size:9.0pt'> attribute.</span></p>
    </td>
   </tr>
   <tr>
    <td width="2%" valign=top style='width:2.52%;border-top:none;border-left:
    solid black 1.0pt;border-bottom:solid black 1.0pt;border-right:none;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.64%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="24%" valign=top style='width:24.68%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal style='margin-top:3.0pt;page-break-after:avoid'><span
    style='font-size:9.0pt;font-family:"Courier New"'><dl dfn-type="element-attr" dfn-for="EventStream">@<dfn>timescale</dfn></dl></span></p>
    </td>
    <td width="13%" valign=top style='width:13.84%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=center style='text-align:center'><span
    style='font-size:9.0pt'>O</span></p>
    </td>
    <td width="56%" valign=top style='width:56.32%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal style='margin-top:3.0pt;page-break-after:avoid'><span
    style='font-size:9.0pt'>specifies the timescale in units per seconds to be
    used for the derivation of different real-time duration values in the </span><b><span
    style='font-size:9.0pt;font-family:"Courier New"'>Event</span></b><span
    style='font-size:9.0pt'> elements.</span></p>
    <p class=MsoNormal align=left style='text-align:left'><span
    style='font-size:9.0pt'>If not present on any level, it shall be set to 1.</span></p>
    </td>
   </tr>
   <tr>
    <td width="2%" valign=top style='width:2.52%;border-top:none;border-left:
    solid black 1.0pt;border-bottom:solid black 1.0pt;border-right:none;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.64%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="24%" valign=top style='width:24.68%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal style='margin-top:3.0pt;page-break-after:avoid'><span
    style='font-size:9.0pt;font-family:"Courier New"'><dl dfn-type="element-attr" dfn-for="EventStream">@<dfn>presentationTimeOffset</dfn></dl></span></p>
    </td>
    <td width="13%" valign=top style='width:13.84%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=center style='text-align:center'><span
    style='font-size:9.0pt'>OD</span></p> <p class=MsoNormal align=left style='text-align:left'><span
    style='font-size:9.0pt'>Default: 0</span></p>
    </td>
    <td width="56%" valign=top style='width:56.32%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal style='margin-top:3.0pt;page-break-after:avoid'><span
    style='font-size:9.0pt'>specifies the presentation time offset of this Event Stream that aligns with the start of the Period. Any Event contained in this Event Stream is mapped to the Period timeline by using the Event presentation time corrected by the value of the presentation time offset.</span></p>
    <p class=MsoNormal align=left style='text-align:left'><span
    style='font-size:9.0pt'>The value of the presentation time offset in seconds is the division of the value of this attribute and the value of the @timescale attribute.</span></p>
    </td>
   </tr>
   <tr>
    <td width="2%" valign=top style='width:2.52%;border-top:none;border-left:
    solid black 1.0pt;border-bottom:solid black 1.0pt;border-right:none;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.64%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="24%" valign=top style='width:24.68%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal style='margin-top:3.0pt;page-break-after:avoid'><b><span
    style='font-size:9.0pt;font-family:"Courier New"'><{Event}></span></b></p>
    </td>
    <td width="13%" valign=top style='width:13.84%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=center style='margin-top:3.0pt;text-align:center;
    page-break-after:avoid'><span style='font-size:9.0pt'>0 ... N</span></p>
    </td>
    <td width="56%" valign=top style='width:56.32%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:9.0pt'>specifies one event.
    For details see Table 5.31. </span></p>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:9.0pt'>Events in Event
    Streams shall be ordered such that their presentation time is
    non-decreasing.</span></p>
    </td>
   </tr>
   <tr>
    <td width="100%" colspan=5 valign=top style='width:100.0%;border:solid black 1.0pt;
    border-top:none;padding:0in 5.4pt 0in 5.4pt'>
    <p class=TH align=left style='margin-bottom:0in;margin-bottom:.0001pt;
    text-align:left'><span style='font-size:9.0pt'>Legend:</span></p>
    <p class=TH align=left style='margin-top:0in;margin-right:0in;margin-bottom:
    0in;margin-left:.25in;margin-bottom:.0001pt;text-align:left'><span
    style='font-size:9.0pt;font-weight:normal'>For attributes: M=Mandatory,
    O=Optional, OD=Optional with Default Value, CM=Conditionally Mandatory.</span></p>
    <p class=TH align=left style='margin-top:0in;margin-right:0in;margin-bottom:
    0in;margin-left:.25in;margin-bottom:.0001pt;text-align:left'><span
    style='font-size:9.0pt;font-weight:normal'>For elements:
    &lt;minOccurs&gt;...&lt;maxOccurs&gt; (N=unbounded)</span></p>
    <p class=TH align=left style='margin-top:0in;margin-right:0in;margin-bottom:
    3.0pt;margin-left:0in;text-align:left'><span style='font-size:9.0pt;
    font-weight:normal'>Elements are </span><span style='font-size:9.0pt;
    font-family:"Courier New"'>bold</span><span style='font-size:9.0pt;
    font-weight:normal'>; attributes are non-bold and preceded with an @.</span></p>
    </td>
   </tr>
  </table>
  <p class=MsoNormal><span lang=DE-AT>&nbsp;</span></p>
  <p class=MsoNormal><span lang=DE-AT>&nbsp;</span></p>
  <table class=MsoNormalTable border=1 cellspacing=0 cellpadding=0 width="99%"
   style='width:99.2%;border-collapse:collapse;border:none'>
   <tr>
    <td width="34%" colspan=5 valign=top style='width:34.44%;border:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=TableCell><b>Element or Attribute Name</b></p>
    </td>
    <td width="14%" valign=top style='width:14.16%;border:solid black 1.0pt;
    border-left:none;padding:0in 5.4pt 0in 5.4pt'>
    <p class=TableCell style='margin-top:3.0pt;page-break-after:avoid'><b>Use</b></p>
    </td>
    <td width="51%" valign=top style='width:51.4%;border:solid black 1.0pt;
    border-left:none;padding:0in 5.4pt 0in 5.4pt'>
    <p class=TableCell style='margin-top:3.0pt;page-break-after:avoid'><b>Description</b></p>
    </td>
   </tr>
   <tr>
    <td width="2%" valign=top style='width:2.54%;border-top:none;border-left:
    solid black 1.0pt;border-bottom:solid black 1.0pt;border-right:none;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.54%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.54%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="26%" colspan=2 valign=top style='width:26.8%;border-top:none;
    border-left:none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><b><span style='font-size:9.0pt;font-family:"Courier New"'><dfn element>Event</dfn></span></b></p>
    </td>
    <td width="14%" valign=top style='width:14.16%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=center style='text-align:center'><b><span
    style='font-size:9.0pt'>&nbsp;</span></b></p>
    </td>
    <td width="51%" valign=top style='width:51.4%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:9.0pt'>specifies an event
    and contains </span><span style='font-size:9.0pt'>the message of the event,
    formatted as a string. The content of this element depends on the event
    scheme.</span><span style='font-size:9.0pt'> </span></p>
    </td>
   </tr>
   <tr>
    <td width="2%" valign=top style='width:2.54%;border-top:none;border-left:
    solid black 1.0pt;border-bottom:solid black 1.0pt;border-right:none;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.54%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.54%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.62%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="24%" valign=top style='width:24.18%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal style='margin-top:3.0pt;page-break-after:avoid'><span
    style='font-size:9.0pt;font-family:"Courier New"'><dl dfn-type="element-attr" dfn-for="Event">@<dfn>presentationTime</dfn></dl></span></p>
    </td>
    <td width="14%" valign=top style='width:14.16%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=center style='margin-top:3.0pt;text-align:center;
    page-break-after:avoid'><span style='font-size:9.0pt'>OD<br>
    default: 0</span></p>
    </td>
    <td width="51%" valign=top style='width:51.4%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=left style='text-align:left'><span
    style='font-size:9.0pt'>specifies</span><span style='font-size:9.0pt'> the
    presentation time of the event relative to the start of the Period.</span></p>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:9.0pt'>The value of the
    presentation time in seconds is the division of the value of this attribute
    and the value of the </span><span style='font-size:9.0pt;font-family:"Courier New"'>@timescale</span><span
    style='font-size:9.0pt'> attribute.</span></p>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:9.0pt'>If not present, the
    value of the presentation time is 0.</span></p>
    </td>
   </tr>
   <tr>
    <td width="2%" valign=top style='width:2.54%;border-top:none;border-left:
    solid black 1.0pt;border-bottom:solid black 1.0pt;border-right:none;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.54%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.54%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.62%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="24%" valign=top style='width:24.18%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal style='margin-top:3.0pt;page-break-after:avoid'><span
    style='font-size:9.0pt;font-family:"Courier New"'><dl dfn-type="element-attr" dfn-for="Event">@<dfn>duration</dfn></dl></span></p>
    </td>
    <td width="14%" valign=top style='width:14.16%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=center style='text-align:center'><span
    style='font-size:9.0pt'>O</span></p>
    </td>
    <td width="51%" valign=top style='width:51.4%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:9.0pt'>specifies</span><span
    style='font-size:9.0pt'> the presentation duration of the event.</span></p>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:9.0pt'>The value of the
    duration in seconds is the division of the value of this attribute and the
    value of the </span><span style='font-size:9.0pt;font-family:"Courier New"'>@timescale</span><span
    style='font-size:9.0pt'> attribute.</span></p>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:9.0pt'>If not present, the
    value of the duration is unknown.</span></p>
    </td>
   </tr>
   <tr>
    <td width="2%" valign=top style='width:2.54%;border-top:none;border-left:
    solid black 1.0pt;border-bottom:solid black 1.0pt;border-right:none;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.54%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.54%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.62%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="24%" valign=top style='width:24.18%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt;font-family:"Courier New"'></span><span
    style='font-size:8.0pt;font-family:"Courier New"'>@<dl dfn-type="element-attr" dfn-for="Event"><dfn>id</dfn></dl></span><span
    style='font-size:8.0pt'></span></p>
    </td>
    <td width="14%" valign=top style='width:14.16%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=center style='margin-top:3.0pt;text-align:center;
    page-break-after:avoid'><span style='font-size:9.0pt'>O</span></p>
    </td>
    <td width="51%" valign=top style='width:51.4%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:9.0pt'>specifies an
    identifier for this instance of the event.&nbsp; Events with equivalent
    content and attribute values in the </span><b><span style='font-size:9.0pt;
    font-family:"Courier New"'>Event</span></b><span style='font-size:9.0pt'>
    element shall have the same value for this attribute. </span></p>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:8.0pt'>The scope of the @id  for each Event is with the same </span><span
    style='font-size:8.0pt;font-family:"Courier New"'>@schemeIdURI</span><span
    style='font-size:8.0pt'> and </span><span style='font-size:8.0pt;
    font-family:"Courier New"'>@value</span><span style='font-size:8.0pt'>
    pair.</span></p>
    </td>
   </tr>
   <tr>
    <td width="2%" valign=top style='width:2.54%;border-top:none;border-left:
    solid black 1.0pt;border-bottom:solid black 1.0pt;border-right:none;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.54%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.54%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.62%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="24%" valign=top style='width:24.18%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt;font-family:"Courier New"'><dl dfn-type="element-attr" dfn-for="Event">@<dfn>contentEncoding</dfn></dl></span></p>
    </td>
    <td width="14%" valign=top style='width:14.16%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=center style='margin-top:3.0pt;text-align:center;
    page-break-after:avoid'><span style='font-size:9.0pt'>O</span></p>
    </td>
    <td width="51%" valign=top style='width:51.4%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:9.0pt'>specifies if the information in the body and the information in the @messageData is encoded.</span></p>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:8.0pt'>If present, the following values are possible:</span></p>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:8.0pt'>
"base64" the content is encoded as described in IETF RFC 4648 prior to adding it to the field. </span></p>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:8.0pt'>
If this attribute is present, the DASH client is expected to decode the message data and only provide the decoded message to the application.</span></p>
    </td>
   </tr><tr>
    <td width="2%" valign=top style='width:2.54%;border-top:none;border-left:
    solid black 1.0pt;border-bottom:solid black 1.0pt;border-right:none;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.54%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.54%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="2%" valign=top style='width:2.62%;border:none;border-bottom:
    solid black 1.0pt;padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt'>&nbsp;</span></p>
    </td>
    <td width="24%" valign=top style='width:24.18%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal><span style='font-size:9.0pt;font-family:"Courier New"'><dl dfn-type="element-attr" dfn-for="Event">@<dfn>messageData</dfn></dl></span></p>
    </td>
    <td width="14%" valign=top style='width:14.16%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=center style='margin-top:3.0pt;text-align:center;
    page-break-after:avoid'><span style='font-size:9.0pt'>O</span></p>
    </td>
    <td width="51%" valign=top style='width:51.4%;border-top:none;border-left:
    none;border-bottom:solid black 1.0pt;border-right:solid black 1.0pt;
    padding:0in 5.4pt 0in 5.4pt'>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:9.0pt'>specifies the value
    for the event stream element. The value space and semantics must be defined
    by the owners of the scheme identified in the </span><span
    style='font-size:9.0pt;font-family:"Courier New"'>@schemeIdUri</span><span
    style='font-size:9.0pt'> attribute.</span></p>
    <p class=MsoNormal align=left style='margin-top:3.0pt;text-align:left;
    page-break-after:avoid'><span style='font-size:9.0pt'>NOTE: this attribute
    is an alternative to specifying a complete XML element(s) in the Event. It
    is useful when an event leans itself to a compact string representation</span></p>
    </td>
   </tr>
   <tr>
    <td width="100%" colspan=7 valign=top style='width:100.0%;border:solid black 1.0pt;
    border-top:none;padding:0in 5.4pt 0in 5.4pt'>
    <p class=TH align=left style='margin-bottom:0in;margin-bottom:.0001pt;
    text-align:left'><span style='font-size:9.0pt'>Legend:</span></p>
    <p class=TH align=left style='margin-top:0in;margin-right:0in;margin-bottom:
    0in;margin-left:.25in;margin-bottom:.0001pt;text-align:left'><span
    style='font-size:9.0pt;font-weight:normal'>For attributes: M=Mandatory,
    O=Optional, OD=Optional with Default Value, CM=Conditionally Mandatory.</span></p>
    <p class=TH align=left style='margin-top:0in;margin-right:0in;margin-bottom:
    0in;margin-left:.25in;margin-bottom:.0001pt;text-align:left'><span
    style='font-size:9.0pt;font-weight:normal'>For elements:
    &lt;minOccurs&gt;...&lt;maxOccurs&gt; (N=unbounded)</span></p>
    <p class=TH align=left style='margin-top:0in;margin-right:0in;margin-bottom:
    3.0pt;margin-left:0in;text-align:left'><span style='font-size:9.0pt;
    font-weight:normal'>Elements are </span><span style='font-size:9.0pt;
    font-family:"Courier New"'>bold</span><span style='font-size:9.0pt;
    font-weight:normal'>; attributes are non-bold and preceded with an @.</span></p>
    </td>
   </tr>
  </table>
  <p class=MsoNormal></p>
  </td>
 </tr>
</table>
  <figcaption>Table 2: MPD Event elements</figcaption>
</figure>


As is shown in Figure 3, each MPD Event has three associated timing
parameters along the media timeline:

1.  The PeriodStart Time (<var>AT</var>) of the Period element containing the EventStream element.

2.  Event Start Time (<var>ST</var>): the moment in the media timeline that a given MPD Event
    becomes active and can be calculated from the attribute <{Event@presentationTime}>.

3.  Event duration (<var>DU</var>): the duration for which the event is active that
    can be calculated from the attribute <{Event@duration}>.

Note that the first parameter is inherited from the Period containing
the Events and only the 2<sup>nd</sup> and 3<sup>rd</sup> parameters are
explicitly included in the <{Event}> element. Each <{EventStream}> also
has <{EventStream/timescale}> to scale the above parameters.

Figure 3 demonstrates these parameters in the media timeline.
<figure>
  <img src="Images/mpdeventtiming.png" />
  <figcaption>Figure 3: MPD events timing model
</figcaption></figure>


The <var>ST</var> of an MPD event can be
calculated using values in its <{EventStream}> and <{Event}> elements:

<figure>

  $$ST = PeriodStart +  
  \frac{<{EventStream/presentationTimeOffset}>}{<{EventStream/timescale}>} + 
  \frac{<{Event/presentationTime}>}{<{EventStream/timescale}>}$$
  <figcaption>Equation 2: Event Start Time of MPD event
</figcaption></figure>

In this document, we use the following common variable names instead of some of above variables to harmonize parameters between Inband events, MPD events, and timed metadata samples:

- <var>scheme_id</var> = <{EventStream/schemeIdUri}>
- <var>value</var> = <{EventStream/value}>
- <var>timescale</var> = <{EventStream/timescale}>
- <var>duration</var> = <{Event/duration}>
- <var>id</var> = <{Event/id}>
- <var>message_data</var> = decode64(<{Event/messageData}>)

In which decode64() function is:
<figure>

  $$decode64(x) = \begin{cases}
x\space\qquad\qquad\qquad\qquad\qquad \space \space \space \space   @contentEncoding\space Not \space Present\\ 
base64 \space decoding \space of \space (x) \qquad @contentEncoding \space  = \space base64
\end{cases}
$$
  <figcaption>Equation 3: decode64 function 
</figcaption></figure>

Note that the DASH client shall Base64 decode the <{Event/messageData}> value if the received <{Event/contentEncoding}> value is base64.

## Timed metadata sample timing model ## {#timed-metadata-timing}

Figure 4 shows the timing model for a given timed metadata sample.
<figure>
  <img src="Images/timedmetadataeventtiming.png" />
  <figcaption>Figure 4: Timing parameters of a timed metadata sample on the media timeline
</figcaption></figure>

As shown in this figure, the metadata sample timing including metadata
sample presentation time (<var>ST</var>) and metadata sample duration (<var>DU</var>). Also
one or multiple metadata samples are included in a segment with Segment start time (<var>AT</var>).

Note that the metadata sample duration can not go beyond segment
duration, i.e. to the next segment. In the case of [=CMAF=], the same
constraints is maintained for CMAF Chunks.

In this document, we use the following common variable names instead of some of above variables to harmonize parameters between Inband events, MPD events, and timed metadata samples:

- <var>scheme_id</var> = <dfn>timed metadata track URI</dfn>
- <var>timescale</var> = <dfn>timed metadata track timescale</dfn> in mdhd box.
- <var>ST</var> = <dfn>timed metadata sample presentation time</dfn>
- <var>duration</var> = <dfn>timed metadata sample duration</dfn>
- <var>message_data</var> = <dfn>timed metadata sample data in mdat</dfn>

# Events and timed metadata sample dispatch timing modes # {#event-metadata-dispatch}

Figure 5 shows two possible dispatch timing models for DASH events and timed metadata samples.
<figure>
  <img src="Images/eventtimedmetadatadispatchmodes.png" />
  <figcaption>Figure 5: The Application events and timed metadata dispatch modes
</figcaption></figure>

In this figure, two modes are shown:

1. <dfn>On-receive</dfn> Dispatch Mode: Dispatching at <var>AT</var> or earlier. Since the segment carrying an emsg/metadata sample has to be parsed before (or assuming zero decode/rendering delay as the latest at) <var>AT</var> on the media timeline, the event/metadata sample shall be dispatched at this time or before to Application in this mode. Application has a duration of <var>ST</var>-<var>AT</var> for preparing for the event. In this mode, the client doesn’t need to maintain states of Application events or metadata samples either. Application may have to maintain the state for any event/metadata sample, its <var>ST</var> and  <var>DU</var>, and monitor its activation duration, if it needs to. Application also needs to schedule each event/sample at its <var>ST</var>, so it must be time-aware to properly make use of these timing parameters.

2. <dfn>On-start</dfn> Dispatch Mode: Dispatching exactly at <var>ST</var>, which is the start/presentation time of the event/metadata sample. The DASH player shall calculate the <var>ST</var> for each parsed event/metadata sample and dispatch the <var>message_data</var> at this exact moment. In this mode, since Application receives the event/sample at its start/presentation time, it needs to act on the received data right away, i.e. no advanced notice is given to Application in this mode. Application however may not need to maintain a state for the events and timed metadata samples, if the durations and/or the sequence and order of events/samples are not important to Application. Depending on the nature, meaning and relationship between different event instances/metadata samples, Application may need to maintain the state for them.

## The Dispatch Processing Model ## {#dispatch-processing}

### Prerequisite ### {#dispatch-prerequisite}

Application subscribes to specific event stream as described in [[#prose-event-API]].

The processing model varies depending on <var>dispatch_mode</var>. 

The DASH player shall follow the processing model outlined in this section.

The DASH player shall set up an [=Active Event Table=] for each subscribed
<var>scheme_uri</var>/(<var>value</var>) in the case of <var>dispatch_mode</var> = <var>on_start</var>. <dfn>Active Event Table</dfn> maintains a single list of emsg’s <var>id</var> that have been dispatched.

### Common process  ### {#dispatch-common-process}
The DASH player shall implement the following process:

1. Parse the emsg/timed metadata sample and retrieve <var>scheme_uri</var>/(<var>value</var>).

2. If Application is not subscribed to the <var>scheme_uri</var>/(<var>value</var>) pair, end the processing of this emsg.

### [=On-receive=] processing   ### {#on-receive-proc}
The DASH player shall implement the following process when <var>dispatch_mode</var> = <var>on_receive</var>:
- Dispatch the event/timed metadata, including <var>ST</var>, <var>id</var>, <var>DU</var>, <var>timescale</var> and <var>message_data</var> as described in [[#prose-event-API]].

### [=On-start=] processing  ### {#on-start-proc}
The DASH player shall implement the following process when <var>dispatch_mode</var> = <var>on_start</var>:
1. Derive the event instance/metadata sample's <var>ST</var> 

2. If the current presentation time value is smaller than C, then go to Step 5.

3. Derive the ending time <var>ET</var>= <var>ST</var> + <var>DU</var>.

4. If  the current presentation time value is greater than <var>ET</var>, then end processing.

5. In the case of event: Compare the event's <var>id</var> with the entries of [=Active Event Table=] of the same <var>scheme_uri</var>/(<var>value</var> pair:
    - If an entry with the identical <var>id</var> value exists, end processing;
     - If not, add emsg’s <var>id</var> to the corresponding [=Active Event Table=].

6. Dispatch the event/metadata <var>message_data</var> at time <var>ST</var>, or immediately if current presentation time is larger then <var>ST</var>,  as described in [[#prose-event-API]]. 

## The event/metadata buffer model ## {#event-metadata-buffer-model}

Along with the media samples, the event instances and timed metadata samples
are buffered. The event/metadata buffer should be managed with same
scheme as the media buffer, i.e. as long as a media sample exists in the
media buffer, the corresponding events and/or metadata samples should be
maintained in the event/metadata buffer.

# Prose description of APIs # {#prose-event-API}

The event/timed metadata API is an interface defined between a “DASH player” as defined in DASH-IF, or a “DASH client” as defined in 3GPP TS 26.247 or ISO/IEC 23009-1 and a device application in the exchange of subscription data and dispatch/transfer of matching DASH Event or timed metadata information between these entities. The Event/timed metadata API is shown at Figure 1.

Note: In this document, the term "DASH player" is used.

The description of the API below is strictly functional, i.e. implementation-agnostic, is intended to be employed for the specification of the API in Javascript for the dash.js open source DASH player, and in IDL such as the OMG IDL or WebIDL. For example, the subscribeEvent() method as defined below may be mapped to the existing **on(type,listener,scope)** method as defined for the dash.js under **MediaPlayerEvents**.

The state diagram of the DASH player associated with the API is shown below in Figure 6:
<figure>
  <img src="Images/eventsubscriptionstatediagram.png" />
  <figcaption>Figure 6: State Diagram of the DASH player for the event/timed metadata API.
</figcaption></figure>

The scope of the above state diagram is the entire set of
applicable events/timed metadata streams being subscribed/unsubscribed, i.e. it is not indicating the state model of the DASH player in the context of a single Event/timed metadata stream subscription/un-subscription.

The application subscribes to the reception of the desired event/timed metadata and associated information by the **subscribeEvent()** method. The parameters to be passed in this method are:

  - <var>app_id</var> – (Optional) A unique ID for the Application subscribing to data dispatch from the DASH player. Depending on the platform/implementation this identifier may be used by the DASH player to maintain state information.

  - <var>scheme_uri</var> – A unique identifier scheme for the associated DASH Event/metadata stream of interest to the Application. This string may use a URN or a URL syntax, and may correspond to either an MPD Event, an inband Event, or a timed metadata stream identifier. The <var>scheme_uri</var> may be formatted as a regular expression (regex). If a value of NULL is passed for <var>scheme_uri</var>, then the application subscribes to all existing event and metadata schemes described in the MPD. In this case, the value of <var>value</var> is irrelevant.

  - <var>value</var> – A value of the event or timed metadata stream within the scope of the above <var>scheme_uri</var>, optional to include. When not present, no default value is defined – i.e., no filtering criterion is associated with the Event scheme identification.

  - <var>dispatch_mode</var> – Indicates when the event handler function identified in the <var>callback_function</var> argument should be called:
    
      - <var>dispatch_mode</var> = <var>on_receive</var> – provide the event/timed metadata sample data to the Application as soon as it is detected by DASH player;
    
      - <var>dispatch_mode</var> = <var>on_start</var> – provide the event/timed metadata sample data to the App at the start time of Event message or at the presentation time of timed metadata sample.

  - <var>callback_function</var> – the name of the function to be (asynchronously) called for an event corresponding to the specified
    <var>scheme_uri</var>/(<var>value</var>). The callback function is invoked with the arguments described below.

Upon successful execution of the event/timed metadata subscription call
(for which the DASH player will return a corresponding
acknowledgment), the DASH player shall monitor the source of
potential Event stream information, i.e., the MPD or incoming DASH
Segments, for matching values of the subscribed <var>scheme_uri</var>/(<var>value</var>). The parentheses around value is because this parameter may be absent in the event/timed metadata subscription call. When a matching event/metadata sample is detected, the DASH player invokes the function specified in the callbackFunction argument with the following parameters. It should additionally provide to the Application the current presentation time at the DASH player when performing the dispatch action. The parameters to be passed in this method are shown in Table 3 below:

<figure>
<table class=MsoTableGrid border=1 cellspacing=0 cellpadding=0
 style='border-collapse:collapse;border:none'>
 <tr>
  <td valign=top style='width:800pt;border:solid #FFFFFF 1pt;
  padding:0in 5.4pt 0in 5.4pt'>
<table class="MsoNormalTable" border="1" cellspacing="0" cellpadding="0" width="99%" style="width: 99.2%; border-collapse: collapse; border: none;">
<thead>
<tr class="header" style="height: 41px;">
<th  style="border-top: 1pt solid black; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 41px; text-align: left; vertical-align: middle; width: 25%;">MPD event</th>
<th style="border-top: 1pt solid black; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 41px; text-align: left; vertical-align: middle; width: 25%;">Inband emsg</th>
<th style="border-top: 1pt solid black; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 41px; text-align: left; vertical-align: middle; width: 25%;">Metadata</th>
<th style="width: 10%; border-top: 1pt solid black; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 41px; text-align: left; vertical-align: middle;">Data Type</th>
<th  valign="top" style="border-top: 1pt solid black; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 41px; text-align: center; vertical-align: middle; width: 40px;">&lsquo;[=On-receive=]&rsquo;</th>
<th  valign="top" style="padding: 0in 5.4pt; border: 1pt solid black; height: 41px; text-align: center; vertical-align: middle; width: 40px;">&lsquo;[=On-start=]&rsquo;</th>
</tr>
</thead>
<tbody>
<tr class="even" style="height: 21px;">
<td  style="width: 7.85865%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;">&lt;{EventStream/schemeIdUri}&gt;</td>
<td style="width: 10.8914%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;">[=scheme_id_uri=]</td>
<td style="width: 3.125%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;"><span>[=timed metadata track URI=]</span></td>
<td style="width: 3.125%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; text-align: left; height: 21px;"><span>&nbsp;</span></td>
<td align="center" width="2%" valign="top" style="width: 15%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px;">Y</td>
<td align="center" width="2%" valign="top" style="width: 15%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: 1pt solid black; padding: 0in 5.4pt; height: 21px;">Y</td>
</tr>
<tr class="odd" style="height: 21px;">
<td  style="width: 7.85865%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;">&lt;{EventStream/value}&gt;</td>
<td style="width: 10.8914%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;">[=value=]</td>
<td style="width: 3.125%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;"></td>
<td style="width: 3.125%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; text-align: left; height: 21px;"></td>
<td align="center" width="2%" valign="top" style="width: 15%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px;">Y</td>
<td align="center" width="2%" valign="top" style="width: 15%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: 1pt solid black; padding: 0in 5.4pt; height: 21px;">Y</td>
</tr>
<tr class="even" style="height: 21px;">
<td  style="width: 7.85865%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;">&lt;{EventStream/timescale}&gt;</td>
<td style="width: 10.8914%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;">[=timescale=]</td>
<td style="width: 3.125%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;"><span>[=timed metadata track timescale=]</span></td>
<td style="width: 3.125%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; text-align: left; height: 21px;"><span>unsigned int(32)</span></td>
<td align="center" width="2%" valign="top" style="width: 15%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px;">Y</td>
<td align="center" width="2%" valign="top" style="width: 15%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: 1pt solid black; padding: 0in 5.4pt; height: 21px;">N</td>
</tr>
<tr class="odd" style="height: 21px;">
<td  style="width: 7.85865%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;">&lt;{Event/presentationTime}&gt;</td>
<td style="width: 10.8914%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;">[=presentation_time=]</td>
<td style="width: 3.125%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;"><span>[=timed metadata sample presentation time=]</span></td>
<td style="width: 3.125%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; text-align: left; height: 21px;"><span>unsigned int(64)</span></td>
<td align="center" width="2%" valign="top" style="width: 15%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px;">Y</td>
<td align="center" width="2%" valign="top" style="width: 15%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: 1pt solid black; padding: 0in 5.4pt; height: 21px;">N</td>
</tr>
<tr class="even" style="height: 21px;">
<td style="width: 7.85865%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;">&lt;{Event/duration}&gt;</td>
<td style="width: 10.8914%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;">[=event_duration=]</td>
<td style="width: 3.125%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;"><span>[=timed metadata sample duration=]</span></td>
<td style="width: 3.125%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; text-align: left; height: 21px;"><span>unsigned int(32)</span></td>
<td align="center" width="2%" valign="top" style="width: 15%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px;">Y</td>
<td align="center" width="2%" valign="top" style="width: 15%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: 1pt solid black; padding: 0in 5.4pt; height: 21px;">N</td>
</tr>
<tr class="odd" style="height: 21px;">
<td style="width: 7.85865%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;">&lt;{Event/id}&gt;</td>
<td style="width: 10.8914%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;">[=id=]</td>
<td style="width: 3.125%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;"></td>
<td style="width: 3.125%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; text-align: left; height: 21px;"><span>unsigned int(32)</span></td>
<td align="center" width="2%" valign="top" style="width: 15%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px;">Y</td>
<td align="center" width="2%" valign="top" style="width: 15%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: 1pt solid black; padding: 0in 5.4pt; height: 21px;">N</td>
</tr>
<tr class="even" style="height: 21px;">
<td style="width: 7.85865%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;">&lt;{Event/messageData}&gt;</td>
<td style="width: 10.8914%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;">[=message_data()=]</td>
<td style="width: 3.125%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px; text-align: left;"><span>[=timed metadata sample data in mdat=]</span></td>
<td style="width: 3.125%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; text-align: left; height: 21px;"><span>unsigned int(8) x messageSize</span></td>
<td align="center" width="2%" valign="top" style="width: 15%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: none; padding: 0in 5.4pt; height: 21px;">Y</td>
<td align="center" width="2%" valign="top" style="width: 15%; border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: 1pt solid black; padding: 0in 5.4pt; height: 21px;">Y</td>
</tr>
<tr class="even" style="height: 21px;">
<td colspan="6" align="left" width="2%" valign="top" style="border-top: none; border-left: 1pt solid black; border-bottom: 1pt solid black; border-right: 1pt solid black; padding: 0in 5.4pt; height: 21px; width: 55.0001%;">Y= Yes, N= NO, O= Optional</td>
</tr>
</tbody>
</table>
</tr>
</table>
<figcaption>Table 3: Event/timed metadata API parameters and datatypes</figcaption>
</figure>


Note:  In the case of ‘emsg’ version 0, the DASH player is expected to calculate [=presentation_time=] from [=presentation_time_delta=].

In order to remove a listener the **unsubscribeEvent()** function is
called with the following arguments:

  - <var>app_id</var> (Optional)

  - <var>scheme_uri</var> - A unique identifier scheme for the associated DASH
    Event stream of interest to the Application.

  - <var>value</var>

  - <var>callback_function</var>

If a specific listener is given in the <var>callback_function</var> argument, then only that listener is removed for the specified <var>scheme_uri</var>/(<var>value</var>). Omitting or passing null to the <var>callback_function</var> argument would remove all event listeners for the specified <var>scheme_uri</var>/(<var>value</var>).
