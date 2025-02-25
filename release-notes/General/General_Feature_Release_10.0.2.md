---
uid: General_Feature_Release_10.0.2
---

# General Feature Release 10.0.2

> [!NOTE]
> For known issues with this version, refer to [Known issues](xref:Known_issues).

## New features

### DMS core functionality

#### DataMiner Indexing \[ID_13370\]\[ID_13406\]\[ID_13504\]\[ID_13571\]\[ID_13623\] \[ID_13622\]\[ID_13629\]\[ID_13695\]\[ID_13769\]\[ID_13912\]\[ID_14001\]\[ID_14038\] \[ID_16287\]\[ID_16896\]\[ID_16915\]\[ID_16935\]\[ID_16959\]\[ID_17081\]\[ID_17166\] \[ID_17328\]\[ID_17851\]\[ID_18562\]\[ID_18714\]\[ID_19337\]\[ID_19437\]\[ID_19443\] \[ID_19691\]\[ID_20373\]\[ID_20845\]\[ID_20998\]\[ID_21205\]\[ID_21257\]\[ID_21634\] \[ID_22378\]\[ID_22927\]\[ID_23049\]\[ID_23998\]\[ID_24054\]\[ID_24158\]

On DataMiner Agents running Cassandra, it is possible to install a dedicated indexing database (e.g. Elasticsearch). If this so-called Indexing Engine is installed, new search features will now become available in the Alarm Console. Additional features are also being developed that will make use of the Indexing Engine in the future.

##### Indexing system requirements

DataMiner Indexing Engine can only be installed on DataMiner Agents with the following characteristics:

- Operating system: Windows Server 2008 R2 or higher (64-bit).

- Free RAM: At least 10 GB.

    > [!NOTE]
    > This requirement refers to the amount of free RAM in the system, not the total amount of RAM. DataMiner Indexing will always reserve 8 GB of RAM when it is in use, and an additional 2 GB of free RAM must be available to ensure that the system can run correctly.

- Average CPU usage: Lower than 70%.

- Local database type: Cassandra.

- Free hard disk space: Same amount as used by Cassandra.

- Separate hard disk from the one containing the Cassandra database.

    > [!NOTE]
    > At least 20% of the disk must remain free at all times.

- Microsoft .Net version: 4.6 or higher.

- If multiple indexing databases will be used, the latency between those databases must be less than 50 ms.

- The minimum number of nodes required in order to install DataMiner Indexing Engine depends on the number of alarms that occurred in the 24 hours prior to the installation. One node is required for every 30,000 alarms per day in the DMS.

> [!NOTE]
> If not all DMAs in the DataMiner System use Cassandra, the button to start the Indexing Engine installation wizard is unavailable. Other system requirements can be checked in the first step of the wizard.

##### Installing the Indexing Engine

To install the Indexing Engine, go to DataMiner Cube’s System Center, select *Search & Indexing*, click the button *Install Indexing Engine...*, and then follow the wizard.

> [!NOTE]
> The *Search & Indexing* page in System Center is only available for users with the user permission *Modules* > *System configuration* > *Indexing engine* > *Configure*.

Like the wizard for the Cassandra migration, the wizard will first check whether the system requirements are met. If necessary, you can close the wizard again in order to continue later after the necessary changes have been made. In case changes are implemented while the wizard is open, a button is available that allows you to refresh the validation data.

Once the installation process is completed, reconnect to DataMiner to make sure all functionality is available.

Please note the following regarding the indexing database:

- In the wizard, you can select on which DMAs in a DMS you install DataMiner Indexing. You will only be able to start the installation if the required minimum number of Agents in the DMS is selected. This minimum number depends on the total number of DMAs in the system. If there are 3 or less DMAs in the system, all DMAs must be selected, otherwise at least 3 DMAs must be selected.

- When several DMAs in a DataMiner System have an indexing database installed, all indexes in all those indexing databases will be linked. That way, when a user launches a search on one DMA, all indexing databases in the DataMiner System will be queried.

- If an indexing database is installed on a Failover system, a database instance will be installed on each of the DMAs, and both instances will be clustered. Also, when a DMA with an indexing database and a DMA without an indexing database are combined into a Failover system, a new database instance will be created on the latter DMA, and both instances will be clustered. When a DMA is removed from a Failover system, the indexing database instance on that DMA is removed from the cluster.

- During the setup of the DataMiner Indexing installation, a backup path has to be specified. For more information, refer to [Configuring DataMiner Indexing backups](#configuring-dataminer-indexing-backups).

- In a DataMiner System, there must be at least 2 master nodes. By default, the 3 DataMiner Agents with the lowest DataMiner ID will act as master node.

- Alarms in the indexing database are kept in two separate tables, one for active alarms and one for closed alarms.

##### DB.xml Indexing configuration

When the DataMiner Indexing Engine installation is complete, the *Db.xml* file will contain an additional \<DataBase> section with the connection information of the indexing database. The "search" attribute will be set to true.

```xml
<DataBases xmlns="http://www.skyline.be/config/db">
  ...
  <DataBase active="true" search="true" type="Elasticsearch">
    ...
```

> [!NOTE]
>
> - Changes to the settings of an indexing database in DataMiner Cube will take effect immediately and do not require a DataMiner restart. A DataMiner restart is only required when a named database is deleted.
> - When the Indexing Engine has been installed, the file *Indexing.xml* file is also added to the Skyline DataMiner folder, containing the configuration of the engine itself.

##### DBMaintenance.xml and DBMaintenanceDMS.xml Indexing configuration

In *DBMaintenance.xml* and *DBMaintenanceDMS.xml*, each \<TTL> tag can have an \<Indexing> subtag, allowing you to specify an override for the default TTL setting for a table that is part of the indexing database. It is possible to specify "infinite" as the indexing TTL, in which case these data will be kept indefinitely.

##### Configuring DataMiner Indexing backups

Backups of the Indexing database are not included in a DataMiner restore package. Instead, the backups are stored in a fixed location, which must be specified during the installation of DataMiner Indexing. This location is the same for all Indexing servers in the cluster.

A number of restrictions apply for the backup path. For more information on these restrictions, refer to the DataMiner Help.

Once DataMiner Indexing has been installed, it is possible to change the backup path using the *Backup path* parameter on the *Backup* page in System Center.

> [!NOTE]
>
> - After you have changed the path in System Center, it is possible that the UI is temporarily disabled while the Indexing nodes are restarted to implement the change. As such, we recommend to only change the backup path if this is absolutely necessary.
> - Only one Indexing data backup can be made per day.
> - Deleting any files from the backup file location will cause a restore to fail.

##### Indexing features configuration in System Center

Once DataMiner Indexing Engine has been installed, the *Search & Indexing* section in System Center contains an additional *Enable indexing on alarms* option. This option must be enabled in order to use the new Alarm Console features mentioned below.

There is also a button available that can be used to migrate booking data to the Indexing database. For more information, see DataMiner Cube: New Migrate booking data to Indexing Engine wizard [ID_21935] [ID_23674] [ID_24410] [ID_24424].

##### Enhanced search in alarm tab pages

When you open a new alarm tab page in the Alarm Console while connected to a DataMiner Agent that has indexing enabled on alarms, a search box at the top of that alarm tab page will now allow you to search for particular alarms or information events.

You can also right-click text in the Alarm Console while holding the left CTRL key (or a different key depending on the *Mouse word highlighting in Alarm Console* user setting), and select *Search for \<text> in new tab*. This will open a new tab with the text in question filled in in the search box.

Next to the search box, you can select a timespan (default: last 24 hours). When you start typing in the search box, the most relevant suggestions that are returned by the server will be displayed below. If a suggestion is too large to be displayed completely, you will be able to view it completely by hovering the mouse over it. However, if it consists of multiple lines, only the first line will be displayed.

After you press Enter or select a suggestion, the alarms matching your search phrase will be retrieved in batches of 50. If there are more than 50 alarms that match your search phrase, a *More results* button will be displayed at the bottom of the list. If you click that *More results* button, any alarms that were added or changed will blink briefly.

Once the first 50 alarms have been retrieved, a graphical representation of the alarm distribution will also be displayed.

By default, different instances of the same alarm will be combined in a single alarm tree in the search results. If you want them to be displayed separately, disable the *History tracking* checkbox.

In the search box, you can use the following special keywords, followed by a colon (“:”) and a search phrase:

- Severity
- Description
- Parameter_description
- Owner
- Value
- Time of arrival
- Status
- Elementname
- Viewnames
- Parentviews
- Protocolname
- ElementProperty\_\<propertyName>
- Viewproperty\_\<PropertyName>
- ServiceProperty\_\<PropertyName>

For example, if you want to search for alarms associated with elements of which the name resembles “probe”, you can enter “Elementname:probe”.

##### New user permission

In the Users / Groups module, in the category *Modules* > *System configuration* > *Indexing engine*, the following user permission is now available:

- *Configure*: Determines whether the user can make any changes to the Indexing Engine configuration.

##### Indexing log information

Log information about the Indexing Engine can be found in a new “Search” log file in the *Logging* section of *System Center*.

In addition, the system will continuously monitor the connection with the Indexing database. If for some reason a node of the Indexing database goes down, an alarm will be displayed in the Alarm Console.

##### GetIndexCountRequest method

A new *GetIndexCountRequest* method has been added in DataMiner, which can be used to retrieve the number of documents in the indexing database.

This method can for instance be used as follows for a logger table:

```csharp
GetIndexCountRequest<LoggerTableData> message = new GetIndexCountRequest<LoggerTableData>(new LoggerTableDataIndex(503,52,1000));
```

### DMS Security

#### HTML5 apps: External user authentication via SAML \[23905\]

The DataMiner HTML5 apps (e.g. Dashboards, Jobs, etc.) now all support external user authentication via Security Assertion Markup Language (SAML).

### DMS Protocols

#### SLProtocol class refactored to an interface \[ID_23787\]

To allow for easier testing, the SLProtocol class has been refactored to an interface.

#### NT_SNMP_GET (295): Possibility to specify the maximum number of repetitions \[ID_23888\]

When sending an NT_SNMP_GET request from within a QAction, it is now possible to pass along the maximum number of repetitions in the elementInfo array.

See the following example:

```csharp
elementInfo[13] = maxRepetitions;
object[] result = (object[])protocol.NotifyProtocol(295/*NT_SNMP_GET*/, elementInfo, tableOids);
```

elementInfo\[13\] should contain a 32-bit integer value (Int32) of at least 1. If no value is specified, or if the value is 0 or less, then the default number of repetitions (6) will be passed along.

> [!NOTE]
> Apart from the maximum value of an Int32 (2^31-1), there are no constraints as to maximum value you can specify. So, use this option with caution. The higher this value is set, the higher the stress on the network and the device in question will be.

#### DataMiner Mediation Layer: Base protocols and device protocols can now contain value mappings \[ID_24127\]

In base protocols and device protocols linked to base protocols, you can now specify \<ValueMapping> tags inside \<LinkTo> tags.

A \<ValueMapping> tag has two attributes:

- a *remoteValue* attribute that refers to the value in the other protocol, and
- a *value* attribute that refers to the value in the current protocol.

Here is an example of a value mapping defined in a base protocol:

```xml
<Mediation>
  <LinkTo protocol="Skyline Discreet Values" pid="10" ops="/:1000000">
    <ValueMapping remoteValue="-1" value="-2"/>
  </LinkTo>
</Mediation>
```

Here is example of a value mapping defined in a device protocol:

```xml
<Mediation>
  <LinkTo pid="70005">
    <ValueMapping remoteValue="1" value="5"/>
    <ValueMapping remoteValue="2" value="7"/>
  </LinkTo>
</Mediation>
```

> [!NOTE]
>
> - Value mappings can be defined for both single-value and column parameters of type string or double. If you define them for parameters with Interprete type “high nibble”, they will be ignored.
> - In case of read/write parameters, value mappings have to be defined on both. Note, however, that they should not be identical. If a value mapping match is found when reading or writing a parameter, then the *ops* attribute of the *LinkTo* tag will be ignored. However, if no relevant mappings could be found for the value in question, the *ops* attribute will be taken into account. This can prove useful in case of exception values.

### DMS Cube

#### DataMiner Analytics: Behavioral anomaly detection and suggestion events \[ID_15723\]\[ID_15914\]\[ID_15916\]\[ID_15951\]\[ID_15952\]\[ID_15976\]\[ID_16001\]\[ID_16050\]\[ID_16163\]\[ID_17279\]\[ID_17462\]\[ID_19224\]\[ID_24095\]\[ID_24126\]\[ID_24147\]

The DataMiner Analytics software now allows you to configure behavioral anomaly detection and to display suggestion events in the Alarm Console.

The purpose of this new feature is to detect anomalous changes in the behavior of individual parameters in real time.

> [!NOTE]
> Behavioral anomaly detection and suggestion events are only available on DMAs using a Cassandra database. If none of the DMAs in a cluster use Cassandra, the anomaly configuration options are not displayed.

##### Current limitations

- Only possible for trended parameters with numeric values.
- Not possible for partial table parameters.
- Anomaly detection is limited to 100,000 parameters per DataMiner Agent.
- Processing of change points (see below) is limited to 1,000 points per second. If the buffer receives more than 100,000 change points in rapid succession, some of those may be disregarded.

##### Change points

Any change in behavior is called a change point. At present, there are five types of change points:

- **Outlier**: A value that suddenly spikes upwards or downwards, but returns to its previous, normal behavior after a few points.
- **Level shift**: A value that shifts upwards or downwards (similar to an outlier) and then stays at that new level.

  Example: A value fluctuating around 0 which starts fluctuating around 10.

- **Variance change**: A value of which the variance either increases or decreases.

  Example: A series like 0.5, 0.6,-0.5,-0.2, 1,… 5,8,9,-5,-6,-2.1,… indicates a variance increase. At some point, a value that is fluctuating around 0 between 1 and -1, starts fluctuating around 0 between 10 and -10.

- **Trend change**: A value that suddenly starts increasing or decreasing at a rate different from the normal behavior.

  Examples:

  - A value fluctuating around 10 (i.e. a trend slope of 0) which suddenly starts increasing 1 unit per second (i.e. a trend slope of 1).
  - A value fluctuating around a line with slope 1 which suddenly starts fluctuating around a line with slope -1.

- **Unlabeled change**: If a change point cannot be classified as one of the above-mentioned change points, it is considered an unlabeled change.

##### Behavioral anomalies

Some change points are more significant or unexpected than others. The stranger the current change point is compared to past change points, the higher its significance will be.

Of every new change point, its significance is calculated based on its characteristics (change point type, direction, absolute or relative change size, etc.), taking into account the parameter's change point history of the last month. The change points that are considered the most significant, i.e. the most “surprising”, are labeled “anomalous”.

Level shifts which are higher or which have a different direction than previous recent jumps, or which jump to a previously unseen level, will typically be labeled “anomalous”. Similarly, trend or variance changes will be labeled “anomalous” when no earlier trend or variance changes in the same direction appeared during the last weeks.

Currently, no change points of type “outlier” or “unlabeled” will be labeled “anomalous”.

##### Change points visible in trend graphs

On a trend graph, a change point is indicated by a bar below the graph. The length of the bar indicates the approximate time frame in which the change started, the height of the bar indicates the importance of the change, and the color of the bar indicates the severity.

When you hover the mouse pointer over a change point bar, a semi-transparent ribbon will appear over the entire height of the trend graph, showing more information about the change point.

Labels of change points of type “trend change” will indicate the level of increase or decrease in seconds, minutes, hours or days depending on the value. If, for example, the value increases by 0.01 per second (i.e. 0.6 per minute, 36 per hour or 864 per day), the label will show an increase of 36 per hour as it is the smallest amount greater than 1.

##### Trend state icons next to parameters in DATA pages

When you open an element card, each trended parameter on that card gets one of the following trend state icons. When you hover over one of those icons, a popup will open, showing additional information.

| Icon | Description |
|--|--|
| ![trend graph icon](~/release-notes/images/StandardTrendGraphIcon.png) | Standard trend graph icon |
| ![upward arrow icon](~/release-notes/images/ArrowRight60.png) | Upward arrow |
| ![downward arrow icon](~/release-notes/images/ArrowRight120.png) | Downward arrow |
| ![flat arrow icon](~/release-notes/images/ArrowRight.png) | Flat arrow |
| ![upward level shift icon](~/release-notes/images/LevelShiftIncrease.png) ![red upward level shift icon](~/release-notes/images/LevelShiftIncreaseRed.png) | Upward level shift |
| ![downward level shift icon](~/release-notes/images/LevelShiftDecrease.png) ![red downward level shift icon](~/release-notes/images/LevelShiftDecreaseRed.png) | Downward level shift |
| ![upward trend change icon](~/release-notes/images/ArrowTrendChangeUp.png) ![red upward trend change icon](~/release-notes/images/ArrowTrendChangeUpRed.png) | Upward trend change |
| ![downward trend change icon](~/release-notes/images/ArrowTrendChangeDown.png) ![red downward trend change icon](~/release-notes/images/ArrowTrendChangeDownRed.png) | Downward trend change |
| ![variance increase icon](~/release-notes/images/ArrowVarianceChangeUp.png) ![red variance increase icon](~/release-notes/images/ArrowVarianceChangeUpRed.png) | Variance increase |
| ![variance decrease icon](~/release-notes/images/ArrowVarianceChangeDown.png) ![red variance decrease icon](~/release-notes/images/ArrowVarianceChangeDownRed.png) | Variance decrease |
| ![upward outlier icon](~/release-notes/images/ArrowOutlierUp.png) ![red upward outlier icon](~/release-notes/images/ArrowOutlierUpRed.png) | Upward outlier |
| ![downward outlier icon](~/release-notes/images/ArrowOutlierDown.png) ![red downward outlier icon](~/release-notes/images/ArrowOutlierDownRed.png) | Downward outlier |

DataMiner will do the following to select a trend state icon for a particular parameter:

1. From the trend data of the parameter, DataMiner will fetch all change points that occurred during the last X seconds. X being the number of seconds specified in the *arrowWindowLength* parameter, found in *C:\\Skyline DataMiner\\Files\\SLAnalytics.config*. Default value: 3600 seconds.

2. If some of the change points are anomalous, then the following trend state icon is selected:

    - Type = Type of most recent anomalous change point
    - Color = Red

    Example: A red, upwards level shift.

    > [!NOTE]
    > If the most recent anomalous change point is an unlabeled change, then the standard trend graph icon is selected.

3. If none of the change points are anomalous, then the following trend state icon is selected:

    - Type = Type of the most recent change point of which the severity is equal to or greater than 0.5. If there are no change points of which the severity is equal to or greater than 0.5, then see step 4.
    - Color = Black

4. In all other cases, one of the following trend state icons is selected, based on the behavior of the parameter value in the last *arrowWindowLength* seconds: a flat arrow, an upward arrow, a downward arrow, or the standard trend graph icon.

##### Suggestion events

By default, a so-called “suggestion event” is generated whenever an anomalous level shift, trend change or variance change is detected for a particular parameter.

In case of a level shift, for example, the value of the suggestion event will be either “Level increased by X (from Y to Z)” or “Level decreased by X (from Y to Z)”. X will be the size of the jump, Y will be an estimation of the previous level and Z will be an estimation of the new data level.

To view these suggestion events, you can create a new tab in the Alarm Console and select to display suggestion events. This is possible for tabs displaying current alarms, history alarms and alarms in a sliding window.

> [!NOTE]
>
> - Currently, no suggestion events are generated for outliers and unlabeled change points.
> - Suggestion events have severity “Information” and source “Suggestion Engine”.
> - Suggestion events generated to indicate a behavioral anomaly are automatically cleared 2 hours after their creation time or their last update time.

#### Possibility to enable SSL/TLS encryption when creating or editing TCP/IP elements \[ID_23262\]\[ID_23617\]\[ID_23836\]\[ID_23947\]

When creating or editing a TCP/IP element, you can now enable SSL/TLS encryption.

To enable TLS, do the following on every DataMiner Agent in a DataMiner System containing elements that use TLS encryption:

1. In the C:\\Skyline DataMiner\\Certificates folder, place a PKCS12 file with (default) name “server.pfx”, containing the certificates and the private key.

1. Send a ConfigureTLSMessage with the following arguments:

   | Argument        | Description                                                                                                                        |
   |-------------------|----------------------------------------------------------------------------------------------------------------------------------|
   | DataMinerID       | ID of the DataMiner Agent ID.                                                                                                    |
   | EncryptedPassword | Encrypted password that will be used with the certificate in question. This is encrypted using the public key of the connection. |
   | Certificate       | Name of the certificate for which the password is set. Default: server.pfx                                                       |
   | ID                | ID of the certificate for which the password is set. Currently, DataMiner will always use the certificate with ID 0.             |

> [!NOTE]
>
> - DataMiner currently supports all TLS versions up to TLS 1.3 (i.e. all TLS versions supported by OpenSSL 1.1.1). It will negotiate the highest supported TLS version with the client. If the client supports TLS up to version 1.2, DataMiner will use version 1.2.
> - PFX files are not synchronized among the agents in a cluster.
> - When, on a DataMiner Agent, you replace a PKCS12 file, then all elements using the TCP/IP port in question have to be stopped and restarted for the changes to take effect.
> - TLS elements and non-TLS elements sharing the same TCP/IP port is not supported.

#### Redesigned DataMiner Cube layout \[ID_23427\]\[ID_23486\]\[ID_23540\]\[ID_23628\]\[ID_23646\]\[ID_23813\]\[ID_23822\]\[ID_23904\]\[ID_23982\]\[ID_24036\]\[ID_24044\]\[ID_24079\]\[ID_24086\]\[ID_24129\]\[ID_24143\]\[ID_24160\]\[ID_24165\]\[ID_24189\]\[ID_24218\]\[ID_24518\]

The design of DataMiner Cube has been updated to be more user-friendly and more in line with other DataMiner apps. Aside from numerous small layout changes, there have been a number of large changes, as detailed below.

##### Redesigned header bar

- In the header bar, the date and time and the four squares indicating the four "sides" of Cube are no longer displayed by default. A drop-down arrow in the header bar provides quick access to settings that allow you to display these again if you prefer this. These settings are also available on the *Cube* tab of the Cube settings card.

- In the middle of the header bar, there is now a search box. This search box features improved search possibilities compared to the search box that was previously included in the side panel. As soon as you click the search box, a list of suggestions is shown below. Initially, this list shows recent items, but it is updated with search results as soon as you type anything in the box. You can click a suggestion to immediately open the corresponding card, or click *Advanced search* at the bottom of the list to open a complete list of search results in the side panel. This list will stay visible until another tab is selected.

    > [!NOTE]
    > Hidden elements are no longer included in the search results.

- Cube settings, card layout settings, updates, and other options that were previously available via various menus of the header bar are now available in one single menu, which can be accessed via the user icon in the top-right corner.

- If an alarm storm occurs, this is no longer displayed in the header bar. Instead, a button is now displayed at the bottom of the Alarm Console. A tooltip on the button provides more information.

- The alarm banner has been replaced with a notification in the header bar. A new Cube setting is available, which allows you to choose between a wide or compact alarm banner.

##### Redesigned side panel

- The side panel on the left or right side of the Cube UI is now by default displayed as a blue bar containing the *Surveyor*, *Activity*, *Apps* and *Workspace* icons. Clicking these icons opens the corresponding panel.

- The *Activity* panel replaces the former *Recent* tab of the side panel, but functions in the same way as this tab, allowing you to pin items to the top of the list of recent activity.

- The icons in the Surveyor and apps panel have been updated. These are now the most commonly used icons:

  | Icon                                                                  | Description      |
  |-----------------------------------------------------------------------|------------------|
  | ![element icon](~/release-notes/images/CubeXElement.png)              | Element          |
  | ![service icon](~/release-notes/images/CubeXService.png)              | Service          |
  | ![redundancy group icon](~/release-notes/images/CubeXRedunGroup.png)  | Redundancy group |
  | ![SLA icon](~/release-notes/images/CubeXSLA.png)                      | SLA              |
  | ![view icon](~/release-notes/images/CubeXView.png)                    | View             |

  If an alarm is present on an element, service, redundancy group or SLA, this is indicated by a colored circle in the bottom-right corner of the icon. For a view, the entire icon is colored according to the alarm severity.

  However, note that these new icons do not support latch level, aggregation level and split view level indications. As such, a new user setting is available, *Use modern icons*, which can be cleared to use the previous icons again.

##### Redesigned logon screen

- The logon screen now features a minimalist design, showing only the DataMiner logo, the IP of the DMA, the login info and a configuration button. With a button at the bottom of the window, you can switch between the current Windows user and a different user profile. The configuration button provides access to options, the "about" information and logging.

- A number of buttons are now available, based on the status of the logon and the DMA. While you are logging onto a DMA, you can click *Back* to return to the logon screen. If a DMA is upgrading or migrating when you log on, you can click the *Monitor* button to monitor the progress. If a DMA is stopped, you can click *Start*, and if a DMA is offline, you can click *Set online*.

#### DataMiner Analytics: Alarm focus \[23911\]\[ID_24083\]\[ID_24102\]\[ID_24128\]

The DataMiner Analytics software will now by default assign an estimated likelihood, called a focus score, to each active alarm after analyzing the short-term history and current behavior of incoming alarms in real time.

Focus scores are numbers ranging from 0 (completely unexpected) to 1 (fully expected). They are stored in AlarmFocusEvents that are generated by SLAnalytics and cached by SLNet. Each AlarmFocusEvent contains an alarm ID and a likelihood field containing the focus score.

The decision whether an alarm is expected is based on likelihood and frequency:

- Likelihood scores are used to spot daily patterns. If an alarm occurs at the same time every day, it will be assigned a high likelihood value at that time.

    > [!NOTE]
    > Likelihood values are based on UTC time. As a result, when daylight saving time goes into or out of effect, patterns following local time might be off for approximately one week.

- Frequency scores are used to detect parameters that frequently go into and out of alarm, or alarms that persist over a long time.

The focus score that is assigned to an alarm is a combination of likelihood, frequency and severity.

> [!NOTE]
>
> - Currently, every DataMiner Agent is responsible for calculating the focus scores of the alarms it is hosting.
> - Currently, no focus score is assigned to the following types of alarms: Correlation alarms, external alarms and information events. By default, those alarms are assigned a focus score equal to null.

##### New column in Alarm Console: Focus

In the *Active alarms* tab page, for each alarm that can be considered “unexpected”, an icon will be displayed in the new *Focus* column, which is located between the *Icon* column and the *Element name* column.

Also, in the blue footer of the *Active alarms* tab page, you will notice a new focus icon. If you click this icon, the current tab page will only display the alarms with a focus icon in the *Focus* column.

> [!NOTE]
> If an alarm template changes, all alarms of the parameters that were assigned that alarm template will have their focus score reset and will get the focus icon. All historical focus data for those alarms will be lost.

##### In the event of an alarm storm

During an alarm storm, focus scores of persistent alarms will not be updated, but as soon as a storm has ceased, all active alarms will have their focus scores updated with the most recent values.

Also, an information event will be generated when alarm focus calculation goes into and out of alarm storm mode.

##### When will alarm focus values be recalculated from scratch?

Alarm focus values will be calculated for the first time after an upgrade to DataMiner version 10.0.2. They will only be recalculated from scratch when SLAnalytics notices that, for whatever reason, the Alarm Focus database table has been cleared.

When calculating alarm focus values for the first time, or when recalculating them from scratch, SLAnalytics will take into account the alarm history of the last week. The time taken by such a recalculation, will depend on the amount of alarms to be updated.

#### New system-wide ClientSettings.json setting to configure whether Data Display pages should be unloaded from memory when you navigate away from them \[ID_23913\]

In the *ClientSettings.json* file, the new system-wide *commonServer.ui.datadisplay.PageUnloadOnNavigatingAway* setting allows you to configure whether Data Display pages should be unloaded from memory when you navigate away from them.

Default value: False

#### Visual Overview: New option to collapse empty rows and columns in the connectivity chain of a service instance \[ID_23941\]

By adding a data field of type ‘ServiceInstance’ to a shape, it is possible to have the connectivity chain of a service instance drawn automatically in Visual Overview.

When configuring such a shape, from now on, you can use the *CollapseEmptyRowsAndColumns* option to automatically collapse all empty rows and columns in the connectivity chain.

Example:

| Shape data field | Value                            |
|------------------|----------------------------------|
| ServiceInstance  | \[this service\]                 |
| Options          | CollapseEmptyRowsAndColumns=True |

#### Settings: 'Table export column separator' setting replaced by 'CSV separator' setting \[ID_23986\]

The *Table export column separator* setting (on the *User \> Data Display* page of the *Settings* window) has now been replaced by the *CSV separator* setting (on the *User \> Regional* page of the *Settings* window).

The separator you select in this new setting will be used in all CSV files exported from Cube.

> [!NOTE]
>
> - This *CSV separator* setting is a Cube-only setting. When a CSV file export is initiated directly by a DataMiner Agent, this setting will be disregarded.
> - Before you import a CSV file that was exported using a previous version of Cube, make sure to check the separator used in that file.
> - When you copy data to the Windows clipboard, that data will always be delimited by TAB characters, regardless of the delimiter specified in the CSV separator setting.

#### Automation/Correlation/Scheduler - Email action: List of reports to be included now indicates whether a report is a legacy report or a Dashboards app report \[ID_24015\]

When, in Automation, Correlation or Scheduler, you configure an email action, you can specify whether the email message has to include a report. To do so, you select the *Include report or dashboard* checkbox and select a report from the list.

From now on, each report listed in the report selection box will have an icon that indicates whether it is a legacy report or a Dashboards app report.

#### DataMiner Cube will now be built as an AnyCPU application \[ID_24168\]

As from DataMiner X, DataMiner Cube will be built as an AnyCPU application.

ClickOnce StandAlone and MSI versions will run as 64-bit processes on 64-bit systems and as 32-bit processes on 32-bit systems.

> [!NOTE]
> Microsoft Internet Explorer always launches PresentationHost.exe as a 32-bit process. As a result, ClickOnce XBAP versions of Cube will always run as 32-bit processes.

#### New sidebar docking position setting \[ID_24178\]

In the restyled Cube X, the docking position of the sidebar can now be controlled by means of the *User \> Surveyor \> Sidebar docking position* setting.

Default: left

#### System Center: Agent state displayed on Agents page and total number of agents displayed on Agents, Database and Backup pages \[ID_24230\]

In *System Center*,

- the state of every agent in the DataMiner System is now displayed on the *Agents* page, and
- the total number of agents in the DataMiner System is now displayed on the *Agents* page, the *Database* page, and the *Select Agents to back up* window (which appears when you click *Execute backup* on the *Backup* page).

#### Visual Overview - Trend component: Filtering the legend’s element list and collapsing or expanding the legend by default \[ID_24349\]

When a shape is showing a trend component, it is now possible to

- filter the list of elements in the legend based on the value of a property of the Visio shape (see the *Filter* data item in the example below), and
- collapse or expand the legend by default (see the *ParametersOptions* data item in the example below).

Example:

| Shape data field  | Value                                                                |
|-------------------|----------------------------------------------------------------------|
| Component         | Trending                                                             |
| Parameters        | *DmaID:ElementID:ParameterID*         |
| ParametersOptions | CollapseLegend:true                                                  |
| Filter            | Property:*PropertyName=PropertyValue* |

#### Visual Overview: Linking a shape to an alarm via the root alarm ID \[ID_24367\]

From now on, a shape can also be linked to an alarm via the root alarm ID.

To do so, add a shape data field of type “Alarm” to the shape, and set its value to DmaID/RootAlarmID. If the alarm cannot be found, the shape will not be displayed.

If you want a shape linked to an alarm to visualize the root alarm ID, you can add a shape data field of type “Info”, and set its value to “ROOT ALARM ID”.

Example:

| Shape data field | Value             |
|------------------|-------------------|
| Alarm            | DmaID/RootAlarmID |
| Info             | ROOT ALARM ID     |

> [!NOTE]
>
> - If a shape is linked to an alarm, you can now also use the Info keywords in the shape text (enclosed in square brackets). For example: “The value of the alarm is \[value\].”
> - Note that, when you link shapes to alarms, only active alarms can currently be shown.

#### Visual Overview: Filtering an alarm tab by clicking an AlarmSummary shape \[ID_24380\]

By linking a shape to an alarm filter using an *AlarmSummary* data item, you can show statistical information about the alarms that match the filter. From now on, it is also possible to have the alarms displayed in an alarm tab, filtered according to the filter specified in the shape.

To do so, add a data item of type *AlarmTab*, and set its value to “Name=”, followed by the name of an alarm tab. See the example below.

When you click the shape, Cube will open the specified alarm tab (if it has a filter applied) and apply the alarm filter specified in the *AlarmSummary* data item. If the alarm tab specified in the *AlarmTab* data item has no alarm filter applied or if no alarm tab exists with that name, one will first be created.

| Shape data field | Value                                                                |
|------------------|----------------------------------------------------------------------|
| AlarmSummary     | type\|sharedfiltername\|ApplyLinkedViewServiceOrElementFilter\|Alarm |
| AlarmTab         | Name=AlarmTabName                                                    |

#### Visual Overview: Using 'info keywords' in all data items of a shape linked to an alarm \[ID_24485\]

Up to now, there were two ways to have a shape linked to an alarm show information about that alarm:

- Add an *Info* data item and set its value to a particular alarm keyword, and type a “\*” character in the shape text. The “\*” character will then be replaced by the value of the keyword you specified in the *Info* data item.
- Enter one or more alarm keywords (wrapped in square brackets) in the shape text. Those will then be replaced by their corresponding value.

From now on, it is also possible to specify alarm keywords in shape data items other than *Info*.

Example:

| Shape data field | Value                        |
|------------------|------------------------------|
| Alarm            | 10/20/500                    |
| SetVar           | MyVariable:\[root alarm id\] |

> [!NOTE]
>
> - To prevent infinite loops, do not specify alarm keywords in a shape data item of type *Alarm*.
> - Currently, it is not yet possible to use these keywords inside other keywords.

#### Visual Overview: Displaying a Visio page when the shape is not linked to an element, a view or a service \[ID_24507\]

Up to now, it was only possible to display a page of a Visio file associated with an object linked to a shape. To have a page named “Details” of a Visio file associated with an element named “MyElement” displayed in a separate window, for example, you had to add two data items to a shape: one of type *Element* set to “MyElement”, and one of type *VdxPage* set to “Details\|Window”.

From now on, it is also possible to display a page from the current Visio file by only adding a single data item of type *VdxPage* to a shape, without any reference to an element, view or service. This will allow you to also display Visio pages when a shape is linked to e.g. an alarm. See the following example:

| Shape data field | Value           |
|------------------|-----------------|
| VdxPage          | Details\|Window |

> [!NOTE]
> If you do not specify a window type suffix (“Window”, “Popup” or “ToolTip”), the Visio page will be displayed inside the shape.

### DMS Reports & Dashboards

#### Dashboards app: Quick-pick buttons added to time range feed \[ID_23133\]

The time range feed can now be configured to show a list of quick-pick buttons that will allow users to enter a preset time range by clicking a single button.

To configure the list of quick-pick buttons to be shown when users click a time range feed, go to edit mode, select the time range feed, open the *Layout* tab, select *Use quick picks*, and select the buttons to be shown.

#### Dashboards app: Enhanced theme configuration \[ID_23258\]

In the dashboard settings page, which is now named “*Dashboards settings*”, the dashboard theme configuration has been enhanced. On this page, you can create, copy and delete themes. When editing a theme, you can now also mark it as the default theme.

Also, there are now two system themes: “Light” and “Dark”. These are fixed themes that cannot be edited or deleted.

Per dashboard, a theme can be selected in the *Layout* tab, which has now also been restyled. From now on, this tab will only allow you to change the layout of a dashboard after selecting the *Override* option.

> [!NOTE]
> When you save a customized dashboard layout as a new theme, you will be asked to confirm this save operation as this will undo all changes you made to the layout of the dashboard you are editing.

#### Dashboards app: New shortcut menu option to move dashboards \[ID_23839\]

When you right-click the name of a dashboard in the navigation pane, there is now a new option that allows you to move that dashboard to another folder.

Also, that right-click menu has been optimized. The *Duplicate* option has been renamed to *Copy*, and the list of menu options has been restructured.

#### Dashboards app: Line chart component can now visualize resource capacity \[ID_23901\]

When you add a line chart component to a dashboard and drag resources onto it, it will display the resource capacity parameters as a stacked trend chart.

If you then click the chart and select a point in time, the legend will list all bookings for that specific point in time. To clear the list, right-click the chart or close the legend.

The resource capacity parameters displayed in the chart can be grouped by parameter or by resource.

#### Dashboards app: Separator used in CSV exports based on CSV separator setting in Cube \[ID_24161\]

The separator used in the CSV exports is based on the “CSV separator” setting in Cube. If this setting cannot be retrieved, in Internet Explorer the system will fall back to the Windows regional settings, while other browsers will fall back to the local browser settings.

#### Dashboards app: UI adapted to new DataMiner X style \[ID_24171\]

The header, sidebar and login screen of the Dashboards app have now been adapted to the new DataMiner X style.

#### Dashboards app: New 'Clear all' action + settings to pin actions \[ID_24356\]

In the dashboard settings, you can now "pin" actions to the header bar. When they are pinned, actions will be displayed as full buttons in the dashboard header bar, e.g. the *Start editing* button. When they are not pinned, the actions can be accessed via an arrow button in the top-right corner of the dashboard.

If a dashboard contains at least one feed, a new *Clear all* action is now available in the dashboard header, which can be used to clear the selection of the feeds in the dashboard.

It is possible to view this new action even when the dashboard is embedded, if "subheader=true" is added to the URL. However, not that this is only the case for the *Clear all* action.

For example: `http://[DMA IP]/dashboard/#/MyDashboards/dashboard.dmadb?embed=true&subheader=true`

#### Dashboards app: New 'Node edge graph' visualization \[ID_24433\]

A new *Node edge graph* visualization is now available in the *Other* category in the Dashboards app. This visualization can be used to display a service definition as a node edge graph. It can display the graph based on a service data filter, based on a service definition ID, or based on a reservation instance ID. If several of these inputs are specified, the input that was specified last will be used.

Several options are available to fine-tune the layout of the component. You can select whether node IDs and/or interfaces should be displayed, whether zooming is enabled, and which edge style is used.

### DMS Automation

#### Possibility to add, update or clear the script output \[ID_23936\]

Up to now, when a parent script added keys to the script output and then ran a subscript that also added keys to the script output, the keys added by the parent script would be deleted. From now on, the keys added by the parent script will by default no longer be deleted. If you want those keys to be deleted, then have the parent script call the engine.ClearScriptResult() method before starting the subscript.

Up to now, the following engine objects could be used to manipulate the script output:

- engine.AddScriptOutput(string key, string value)

    Adds a key to the script output if it has not yet been added. If the key already exists, an exception will be thrown.

- subScriptOptions.GetScriptResult()

    Returns a copy of the script output of the current script and, if the InheritScriptOutput option is set to “true”, the child scripts. For more information, see below.

From now on, you can also use the following new engine objects:

- engine.AddOrUpdateScriptOutput(string Key, string Value)

    Adds a key to the script output if it has not yet been added. If the key already exists, it will update it with the specified value.

- engine.ClearScriptResult()

    Clears the script output.

- engine.ClearScriptOutput(string key)

    Removes a key from the script output. Returns NULL when the specified key cannot be found.

- engine.GetScriptResult()

    Returns a copy of the script output of the current script and, if the InheritScriptOutput option is set to “true”, the child scripts. For more information, see below.

- engine.GetScriptOutput(string key)

    Returns the script output of the specified key. Returns NULL when the specified key cannot be found.

> [!NOTE]
> When a subscript fails or throws an exception, its script output will still be filled in.

Also, a new InheritScriptOutput script option will now allow you to control whether a parent script will inherit the script output of its subscripts. Default value: true

Example:

```csharp
var scriptOptions = engine.PrepareSubScript("MyScript");
scriptOptions.InheritScriptOutput = true;
scriptOptions.StartScript();
```

#### Interactive Automation scripts: Uploading files from a client computer \[ID_23950\]\[ID_24144\]\[ID_24164\]

In an interactive Automation script, it is now possible to upload files from a client computer.

To allow users to do so, you need to add a file selector control to the script in the following manner:

```csharp
UIBlockDefinition uiBlock = new UIBlockDefinition();
uiBlock.Type = UIBlockType.FileSelector;
uiBlock.DestVar = "varUserUploadedFile";
```

When the UI is then shown using *Engine#ShowUI(...)*, *UIResults* will contain the path to the file:

```csharp
UIResults results = engine.ShowUI(uiBuilder);
string uploadedFilePath = results.GetUploadedFilePath("varUserUploadedFile");
```

When you have selected a file, the actual upload will only start after you click a button to make the script continue (e.g. Close, Next, etc.). Once the upload has started, a *Cancel* option will appear, allowing you to abort the upload operation.

All files uploaded by users will by default be placed in the *C:\\Skyline DataMiner\\TempDocuments* folder, which is automatically cleared at every DataMiner startup.

#### New engine.UnsetFlag method to clear run-time flags \[ID_23961\]

In an Automation script, you can now use the engine.UnsetFlag method to clear the following run-time flags:

- RunTimeFlags.AllowUndef
- RunTimeFlags.NoInformationEvents
- RunTimeFlags.NoKeyCaching

This will allow you to, for instance, perform silent parameter sets. See the following example:

```csharp
public void SetParameterSilent(int pid, object value) {
    // Set the NoInformationEvents flag to disable information events
    _engine.SetFlag(RunTimeFlags.NoInformationEvents);
    // Perform a silent parameter set without triggering an information event
    element.SetParameter(pid, value);
    // Re-enable information events by clearing the NoInformationEvents flag
    _engine.UnsetFlag(RunTimeFlags.NoInformationEvents);
}
```

### DMS Web Services

#### Web Services API v1: 'GetTableForParameterV2' method has new 'as-kpi' table filter option \[ID_23928\]

The GetTableForParameterV2 method now supports filtering based on the following table column KPI options:

- KPIHideWrite
- HideKPI
- HideKPIWhenNotInitialized
- KPIShowDisplayKey
- KPIShowPrimaryKey
- DisableHistogram

To enable KPI option filtering when calling the *GetTableForParameterV2* method, specify the “as-kpi” filter in the *Filters* input field.

### DMS Mobile apps

#### New Monitoring & Control app \[ID_21736\]\[ID_22023\]\[ID_22209\]\[ID_22750\]\[ID_22801\]\[ID_22888\]\[ID_22943\]\[ID_23025\]\[ID_23036\]\[ID_23090\]\[ID_23387\]\[ID_23798\]\[ID_23874\]\[ID_24017\]\[ID_24059\]\[ID_24072\]\[ID_24080\]\[ID_24114\]\[ID_24134\]\[ID_24180\]\[ID_24192\]

The DataMiner HTML5 app has now been replaced by the new Monitoring & Control app, of which the overall look and feels closely resembles that of the newly redesigned Cube X.

- Redesigned header bar:

  - The app title “Monitoring & Control” is now a button that redirects the user to the app’s homepage.
  - Like in Cube X, the Search box has now been moved from the side panel to the middle of the header bar.
  - On the right, there is now one single user icon, which, when clicked, opens a menu that allows users to access to the settings window and the About box.

    Currently, the settings window allows you to specify the default pages for element and view cards.

- A new homepage similar to the Cube X homepage, listing recently used items.
- Redesigned (collapsible) side panel, on which alarm states are now indicated by colored circles.
- Redesigned element, service, view and alarm cards, which can be accessed directly using the following URLs:

  - `http://<DMAIP>/monitoring/element/<DMAID>/<EID>/data/<PAGENAME>`
  - `http://<DMAIP>/monitoring/element/<DMAID>/<EID>/visual/<PAGENAME>`
  - `http://<DMAIP>/monitoring/element/<DMAID>/<EID>/chain/<CHAINNAME>`
  - `http://<DMAIP>/monitoring/view/<VIEWID>/<PAGENAME>`
  - `http://<DMAIP>/monitoring/view/<VIEWID>/visual/<PAGENAME>`
  - `http://<DMAIP>/monitoring/alarm/<DMAID>/<ALARMID>`

> [!NOTE]
> To open this new Monitoring & Control app from the address bar of your internet browser, enter the IP address of the DataMiner Agent, followed by “/monitoring”.
>
> Alternatively, you can go to the landing page of the DataMiner Agent (by entering its IP address), and then click the *Monitoring & Control* button.

#### Legacy Monitoring & Control app: New settings to specify the default page for element, service and view cards \[ID_24017\]

In the *Settings* window of the legacy Monitoring & Control app, it is now possible to specify the default pages for element, service and view cards.

#### Jobs app: UI adapted to new DataMiner X style \[ID_24157\]

The header and login screen of the Jobs app have now been adapted to the new DataMiner X style.

#### New DataMiner landing page \[ID_24239\]

When you open a browser window and enter the IP address or host name of a DataMiner Agent, you are now directed to a new DataMiner landing page (“/root”).

After signing in, you will be presented with a list of apps (e.g. Monitoring, Dashboards, etc.).

Clicking the user menu in the upper-right corner will allow you to open the Tools page, the About page and DataMiner Help.

### DMS Service & Resource Management

#### DataMiner Cube: New Migrate booking data to Indexing Engine wizard [ID_21935] [ID_23674] [ID_24410] [ID_24424]

In DataMiner Cube, a new wizard has been added to the *Search & Indexing* section of *System Center*. This wizard allows you to migrate booking data from the Cassandra database to the Indexing database.

As some booking property names can contain characters that are considered invalid by the Indexing engine, the wizard will ask you to rename certain booking properties before starting the migration. To ensure the correct functionality of the Service & Resource Management module, some properties will be renamed automatically. For example, the *Visual.Background* and *Visual.Foreground* properties will automatically be renamed as *VisualBackground* and *VisualForeground*.

When you have successfully migrated all booking data, the button to start the wizard will disappear from the UI. Also, when the migration has finished, a *Retrieve report* button will allow you to generate a report showing a summary of that migration.

> [!NOTE]
>
> - After migrating the booking data to the Indexing database, make sure to check your Automation scripts and Visio files and adjust the booking property names where necessary.
> - When configuring backup settings in the *Backup* section of *System Center*, a new *Include SRM in backup* option is now available under the *Create a backup of the database* option. Select this option if you want the booking data in the Indexing database to be included in the backup package.
> - An Indexing database takes about twice as much disk space to store booking data compared to a Cassandra database.
> - A number of methods in the ServiceManagerHelper and ResourceManagerHelper classes have been adapted to allow them to manage booking data stored in an Indexing database.

#### Improvements ResourceManagerHelper class for deletion of resources \[ID_24002\]\[ID_24563\]

If there is an error deleting resources using the *ResourceManagerHelper* class, additional feedback is now returned:

- If one or more resources are deleted using the *ResourceManagerHelper* class, and at least one of the resources fails to be deleted, a single error, *ResourceDeleteFailed*, is returned, which now contains a *UsingIDs* property with all the IDs of the resources that failed. Any possible other resources are still successfully deleted.

- If a resource is deleted that is in use or in maintenance mode, the error *ResourceDeleteInUseOrMaintenanceMode* is returned. If the resource is included in scheduled, ongoing or quarantined bookings, the IDs of these bookings are now returned in the *FutureOrOngoingReservationIds* property.

If a resource that is used in past bookings is deleted, the deletion now fails by default, returning the error *ResourceDeleteInUseOrMaintenanceMode* with the boolean property *HasPastBookings* set to true.

In addition, the following new method is now available in the *ResourceManagerHelper* class: `Resource[] RemoveResources(Resource[] resources, bool ignorePastReservations);`

If *ignorePastReservations* is false, this method works in the same way as existing calls to remove resources. If it is set to true, past bookings will be ignored during the deletion checks. This can be used to delete resources that are only used in past bookings; however, note that these bookings may then contain invalid references to resources.

#### DataMiner will now generated an error when it detects a ServiceManager license but no ElasticSearch instance \[ID_24329\]

From now on, a DataMiner Agent will generated the following DataMiner run-time error when it detects a ServiceManager license but no Elasticsearch instance:

*The Service Manager is licensed, but no ElasticSearch database is active on the system. Therefore, Resource Manager and Service Manager will not initialize.*

## Changes in DataMiner 10.0.2

### Enhancements

#### DataMiner Cube: New label indicating new trend group that is being added \[ID_19526\]

When a new trend group is being added in DataMiner Cube, a "\[new\]" label will be added to the trend group name in the pane that lists the existing trend groups, just as this is done for scripts in the Automation app, for example.

#### Services app - Services tab: Enhanced list view column configuration \[ID_23785\]

When you right-click a column in the *Services* tab and click *Add/remove column*, you can also select *More* to open the *Choose details* window. In that window, the way in which to set a column’s type (which, by default, is set to “Text”) has now been enhanced.

To set the type of a column, open the *Column type* selection box, and

- select “Alarm icon” to display the IDs in the column as alarm icons,
- select “Custom icon” to have the column contents replaced by icons,
- select “Color” to have the cells of that column visualized as colored blocks, or
- select “Date” to have the column contents displayed as date values.

#### Dashboards app: Trend statistics component now uses caching \[ID_23853\]

To improve performance, the trend statistics dashboard component now uses caching.

#### DataMiner Cube - Service & Resource Management: Enhanced profile management \[ID_23861\]

A number of enhancements have been made with regard to the management of profile definitions, profile instances and profile parameters.

#### Failover: Additional folders synchronized at initial setup \[ID_23875\]

From now on, at initial Failover setup, the following folders under *C:\\Skyline DataMiner\\* will also be synchronized among the two DataMiner Agents:

- Aggregation
- Configurations\\JSON
- Dashboards
- ProfileManager
- ProtocolFunctionManager
- ProtocolFunctionManager\\SrmProtocolGenerationInfo
- ServiceManager
- VisualData\\Data
- VisualData\\Objects

#### Dashboards app: Tooltips for component quick menu \[ID_23879\]

Tooltips are now available for the quick menu below a dashboard component, so that a user can see what the icons in the quick menu can be used for by hovering the mouse pointer over them.

#### Logging: Log entry type of LDAP notifications changed from 'error' to 'debug' \[ID_23915\]

Up to now, LDAP notifications were of log entry type “error”. From now on, those notifications will be of type “debug”.

#### Dashboards app: Enhanced performance \[ID_23927\]

A number of performance enhancements have been made to the Dashboards app, especially with regard to the use of button panels.

The maxJSONLength setting, for instance, has now been made configurable. Up to now, this setting, which is used when serializing and deserializing WebSocket messages, was always set to 4MiB.

To set maxJSONLength to a specific value, do the following:

1. Open the *Web.config* file located in the *C:\\Skyline DataMiner\\Webpages\\API* folder.

1. Make sure that the *configuration.appSettings* section contains an element similar to the following one:

    ```xml
    <add key="maxJsonLength" value="?10485760?" />
    ```

    Note that the value should be specified in bytes. In the example above, maxJSONLength was set to 10 MiB.

> [!NOTE]
> If no maxJSONLength value is specified in the Web.config file, this setting will be set to 20 MiB by default.

#### Enhanced performance when filtering on values in tables \[ID_23964\]

Due to a number of enhancements, overall performance has increased when filtering on values in tables.

#### Additional server-side security checks when element properties are added, updated or deleted \[ID_23968\]

From now on, an additional server-side security check will be performed when a user adds, updates or deletes an element property.

#### Enhanced performance when requesting spectrum trend data from SLNet \[ID_23971\]

Due to the introduction of a dedicated cache, overall performance has increased when requesting spectrum trend data from SLNet using GetSpectrumTrendTraceTimestampsMessage and GetSpectrumTrendTraceDataMessage.

#### Web Services API v1: Enhanced performance of GetServicesForView method \[ID_23972\]

Due to a number of enhancements, overall performance has increased when calling the *GetServicesForView* method.

#### DataMiner Cube - Visual Overview: No more logging if no highlight direction can be found for connections \[ID_23973\]

When highlighting of DCF connections was configured in a Visio drawing but no specific direction could be found for a connection, up to now, log entries were added to the Cube logging. However, as this could cause many unnecessary log entries, this will now no longer be logged.

#### Services app: Service definition diagram enhancements \[ID_23985\]

In the *Services* app, a number of enhancements have been made to the diagrams that graphically visualize service definitions.

Also, when you embed the Services app in Visual Overview by using a Service Manager component, it is now possible to have that component display node IDs. To do so, add the following option to the *ComponentOptions* shape data field:

| Shape data field | Value              |
|------------------|--------------------|
| ComponentOptions | “ShowNodeIDs=true” |

#### Enhanced performance when reading data from an Elastic database \[ID_23994\]

Due to a number of data serialization enhancements, overall performance has increased when reading data from an Elastic database.

#### IIS URL Rewrite module will now automatically be installed during a DataMiner upgrade \[ID_24022\]

The IIS URL Rewrite module will now automatically be installed during a DataMiner upgrade.

#### Service & Resource Management: Function XML-related enhancements \[ID_24037\]

The following enhancements have been implemented in the Service & Resource Management module:

- The *FunctionDefinition* object now contains a list of *ExportRules*, which contain the export rules defined in the function XML file.

- It is now possible to upload function XML files that have the "broadcast" option defined on an interface.

- When *ProtocolFunctionVersion.ToXml* is called and no *ParameterGroupLink* is defined, the *ParameterGroupLink* property will now no longer be present on an interface in the resulting XML.

- When *ProtocolFunctionVersion.ToXml* is called, the resulting XML will now wrap the contents of the "Graphical" tag in a CData tag.

#### Dashboards app - Group component: Option to have the legend be expanded or collapsed by default \[ID_24046\]

The Group component now has an option that allows you to specify whether its legend should by default be expanded or collapsed.

If you do not set this option, the legend will by default be collapsed.

#### Cassandra service: Restart time-out increased from 10 seconds to 5 minutes \[ID_24084\]

When the Cassandra service goes down, DataMiner automatically attempts to restart the service.

The time-out for the Cassandra service to change from “starting” to “started” has now been increased from 10 seconds to 5 minutes. Also, if a restart attempt fails or times out after 5 minutes, DataMiner will now keep trying indefinitely every 60 seconds.

Logging has also been improved. Each time a restart attempt has failed, an error will now be added to the log.

#### Cassandra service: Startup time-out increased from 10 seconds to 5 minutes \[ID_24088\]

When the Cassandra service is down while DataMiner is starting up, DataMiner will automatically attempt to start the service.

The time-out for the Cassandra service to change from “starting” to “started” has now been increased from 10 seconds to 5 minutes.

- If DataMiner is not able to start the Cassandra service in 5 minutes, it will start up in file offload mode with all elements in an error state.

    > [!NOTE]
    > Starting the Cassandra service by hand requires a full DataMiner restart.

- If DataMiner is able to start the Cassandra service, but not to establish a database connection, DataMiner will try to establish a connection for the number of times specified in the following Db.xml setting:

    ```xml
    <DataBase active="true" local="true" type="Cassandra">
      <ConnectionRetries>60</ConnectionRetries>
    </DataBase>
    ```

#### Service & Resource Management: Enhanced performance when opening the Services app or Profiles app \[ID_24112\]

Due to a number of enhancements, overall performance has increased when opening the Services app or the Profiles app on systems with a large number of service definitions and profile definitions.

#### Dashboards app - Parameter table component: Columns of which the 'disableHistogram' option is set will no longer have a histogram icon in their header \[ID_24131\]

In a parameter table component, from now on, table columns of which the “disableHistogram” option is set will no longer have a histogram icon displayed in their header.

#### DataMiner Cube: Enhanced performance when drawing card breadcrumbs \[ID_24151\]

Due to a number of enhancements, overall performance has increased when drawing the breadcrumbs at the top of a card.

#### SLNetClientTest tool: Failover Agents included when viewing across cluster \[ID_24170\]

In the SLNetClientTest tool, for some messages (e.g. *Diagnostics* > *SLNet* > *StackSizes*) it is possible to right-click the message and select *View across cluster* to send it to the entire DMS. This feature has now been improved to include Failover Agents as well.

#### DataMiner Cube - Automation/Correlation: Width of action dialog boxes will now automatically be adapted to the size of the screen \[ID_24190\]

From now on, when you configure actions in Automation scripts or Correlation rules, the action dialog boxes will automatically adapt their width to the size of the screen.

#### Enhanced view data synchronization \[ID_24197\]

A number of enhancements have been made with regard to the synchronization of view data among the agents in a DataMiner System.

#### Jobs app: Enhanced error handling when adding jobs \[ID_24238\]

Error handling when adding jobs has been enhanced.

#### DataMiner Cube - Services app: Service list will no longer be initialized when Visual Overview only shows the service definition editor \[ID_24242\]

From now on, the services list in the Services app will no longer be initialized when Visual Overview only shows the service definition editor.

#### DataMiner Cube: Enhanced performance when generating authentication tickets for embedded web browsers \[ID_24261\]

Due to a number of enhancements, overall performance has increased when generating authentication tickets for embedded web browsers in apps like Ticketing, Dashboards, Maps, etc.

#### Dashboards app: Updated design rotate button in button panel \[ID_24272\]

The design of the rotate button in a button panel component has been updated.

#### Smart-serial communication over IP: Receive time-out reduced from 100 ms to 15 ms \[ID_24282\]

In case of smart-serial communication over IP, the hard-coded time-out when receiving data has now been reduced from 100 ms to 15 ms.

#### Service & Resource Management: Enhanced performance when saving and synchronizing resources \[ID_24289\]

Due to a number of enhancements, overall performance has increased when saving resources and synchronizing them among the agents in the DataMiner System.

#### DataMiner Maps: Enhanced performance when opening a pop-up balloon \[ID_24292\]

Due to a number of enhancements, overall performance has increased when opening a pop-up balloon.

#### Improved logging when uploaded protocol requires higher DataMiner version \[ID_24298\]

Logging has improved when a protocol is uploaded that requires a higher DataMiner version than the one currently installed in the system. The following error message will now be logged: "DataMiner version is too low to use this protocol. Please check the protocol's Compliancies tag."

#### DataMiner Cube - Visual Overview: Enhanced performance when using 'FollowPathColor' and 'InternalInterfaceHopping' \[ID_24305\]

Due to a number of enhancements, overall performance has increased when using “FollowPathColor” and “InternalInterfaceHopping”, especially when a large number of shapes are linked to elements containing many interfaces and connections.

#### Alarm level linking: Enhanced performance \[ID_24308\]

Due to a number of enhancements, overall alarm level linking performance has increased.

Also, a new table filter option "sort=NONE" has been added. Using this option will prevent SLElement from applying any kind of sorting.

#### DataMiner Cube - Stream Viewer: Buffer size increased to 1 MB \[ID_24330\]

The buffer size of Stream Viewer’s text box has increased from 32 kB to 1 MB.

#### Possibility to show information message in mobile apps + Dashboards app layout update \[ID_24336\]

In the Monitoring, Dashboards and Jobs apps, it is now possible to use the *ShowInformationMessage* method in a script to display a message in the app.

Note that a small change has also been implemented in the Dashboards app. The *New dashboard* button will no longer be displayed in the header bar. Creating a new dashboard remains possible via the context menu of the side panel and via the Dashboards homepage.

#### HTML5 apps: Enhanced icons \[ID_24341\]

All HTML5 apps (Dashboards, Monitoring, Jobs, etc.) now have enhanced icons.

#### Improved error message when bookings overlap \[ID_24346\]

When a booking is saved and there is another booking that overlaps it, previously all overlapping bookings would be returned in the error message. Now the error message will only return the bookings that conflict with the booking that is being saved.

#### Service & Resource Management: Enhanced processing of event messages related to ReservationInstances and ReservationDefinitions \[ID_24350\]

Due to a number of enhancements, the processing of event messages related to ReservationInstances or ReservationDefinitions has improved.

#### Jobs app: Timeline improvement \[ID_24352\]

In the Jobs app, the way items are displayed on the timeline has been improved:

- Items on the timeline now have a minimum width, so that they are displayed even if they have a very short duration.

- If items on the same height are very close to each other, they will be merged into one aggregated item.

- Clicking an item will now always select it. Clicking an aggregated item will select all the items it combines. To deselect an item, keep Ctrl pressed while you click it.

#### No more empty cookie in request header for embedded Chromium browser \[ID_24387\]

If the request header for an embedded Chromium browser has an empty cookie, the cookie will now no longer be included in the request header. This will improve compatibility with web servers that cannot handle an empty cookie field.

#### DataMiner Cube: Enhanced breadcrumbs on service cards \[ID_24408\]

A number of enhancements have been made to the way in which breadcrumbs are displayed when you open a service cards.

#### HTML5 apps: More consistent layout due to shared UI components \[ID_24414\]

The overall layout of the different HTML5 apps has been made more consistent.

#### DataMiner Cube: Enhanced handling of table configuration errors in protocols \[ID_24432\]

From now on, due to a number of enhancements, DataMiner Cube will be able to better handle any table configuration errors when rendering tables.

Also, when such errors are encountered, additional information will be added to the logging.

#### FullFilter can no longer be used when filtering the list of elements from which a direct view retrieves data \[ID_24445\]

When filtering the list of elements from which a direct view retrieves data, from now on, it will no longer be possible to use FullFilter. However, it will still be possible to do so using a VALUE=parameterId==value filter.

#### SLAnalytics: Enhanced alarm template monitoring \[ID_24477\]

Due to a number of enhancements, overall performance of SLAnalytics has increased when monitoring alarm templates.

#### DataMiner Cube - Visual Overview: Improved performance when using \[param:\] placeholder \[ID_24493\]

When a \[param:\] placeholder is used in Visio, the way parameters are retrieved has been improved, resulting in improved performance.

#### New default values for MySQL settings when installing a new DataMiner Agent \[ID_24516\]

When installing a new DataMiner Agent with a MySQL database, the following MySQL settings will now get a new default value:

| Setting          | Value |
|------------------|-------|
| max_connections  | 1000  |
| secure_priv_file | ""    |

#### Web Services API v1: Unused method and arguments removed \[ID_24530\]

The method *GetClusterAsync*, which could not yet be used, has been removed from the Web Services API v1. In addition, a number of fields that also could not yet be used have been removed. The following fields have been removed from the *DMAAutomationScriptSettings* object:

- DebugMode
- AllowUndef
- SupportsBackAndForward
- SkipElementChecks
- SavedFromCube
- SkipSetInfoEvents

The following fields have been removed from the *DMAScatterAxisInfo* object:

- ParameterID
- TableIndex

#### Service & Resource Management: Enhanced performance when updating active function definitions \[ID_24554\]

Due to a number of enhancements, overall performance has increased when updating active function definitions.

#### Annotation web pages now contain an explicit UTF-8 character encoding declaration \[ID_24555\]

From now on, the annotation web pages will contain the following explicit UTF-8 character encoding declaration:

```xml
<meta charset="utf-8">
```

### Fixes

#### DataMiner Cube: Problem with Element migration window \[ID_23237\]

When connected to a DataMiner System with multiple DataMiner Agents, you opened *System Center \> Agents \> Status*, clicked *Migrate*, selected a destination agent and a number of elements, and then clicked either *Migrate* to start migrating the selected elements to the selected agent or *Cancel* to exit the *Element migration* window, in some cases, an exception could be thrown.

#### DataMiner Cube - Alarm template groups: Table column parameters listed in incorrect order \[ID_23675\]

When an alarm template group listed multiple table column parameters, in some cases, those parameters would be sorted incorrectly.

#### Problem with protocol buffer serialization when server and client were running different DataMiner versions \[ID_23679\]\[ID_24035\]

When protocol buffer serialization was being used, a number of issues could occur when a DMA and a client were running different DataMiner versions.

#### DataMiner Cube - Router Control: Problem in case of multiple consecutive matrix updates \[ID_23680\]

In the Router Control app, in some rare cases, an exception could be thrown when a matrix was updated multiple times in rapid succession.

#### DataMiner Cube - Visual Overview: No effect when clicking shape \[ID_23713\]

In some cases, it could occur that clicking a shape had no effect.

#### SLProtocol crash on multi-threaded timers \[ID_23744\]

In some cases, an error could occur in SLProtocol when using multi-threaded timers on an element with debug logging enabled at level 3 or higher.

#### DataMiner Cube: Not possible to select value for alarm template condition based on discrete parameter with dependency \[ID_23789\]

When a condition was added on a parameter in an alarm template and this condition was set to be based on the value of a discrete parameter with a dependency on another parameter, it could occur that no value could be selected. This issue has been resolved. In addition, in the configuration of an alarm template condition, it is now no longer possible to use a wildcard for parameters that can only have discrete values.

#### Dashboards app: Polling issues when WebSocket communication was disabled \[ID_23801\]

When WebSocket communication was disabled for a dashboard, it could occur that the state of views and redundancy groups was only polled every 5 minutes. In addition, it could occur that only the initial active alarms were retrieved.

#### DataMiner Cube - Visual Overview: \[Sum:X,Y,Z\] placeholder did not parse decimal values \[ID_23810\]

In some cases, it could occur that the *\[Sum:X,Y,Z\]* placeholder in Visio did not parse decimal values.

#### Dashboards app - Trend charts: Boundary lines would only be drawn on the first axis \[ID_23837\]

When you added boundary lines to a trend chart, in some cases, the lines would only be drawn on the first axis. From now on, they will be drawn on every visible axis.

#### DataMiner Cube - Alarm Console: Problem when sorting alarms after grouping them \[ID_23845\]

When, in the hamburger menu of the Alarm Console, the *Automatically group according to arrangement* option is disabled, you can group the alarms by right-clicking a column header and selecting *Group by this field*. You can then sort the alarms in the different groups by clicking a column header. In some cases, however, after grouping alarms by a particular column, it was no longer possible to sort alarms by that same column.

#### Problem with SLElement \[ID_23846\]

In some rare cases, an error could occur in the data gateway table thread of SLElement.

#### NotifyDataminer call could contain empty information \[ID_23863\]

In some rare cases, NotifyDataMiner calls would include insufficient information.

#### Duplicated service definition not identical with original \[ID_23870\]

If a service definition was duplicated, it could occur that the interface configuration of the duplicate was not the same as that of the original.

#### Service & Resource Management: Problem when sorting a ListView by a custom property column \[ID_23876\]

When you sorted a bookings list or a services list by a custom property column, in some cases, the sort order would not be remembered. The next time you opened the *Bookings* app or the *Services* app, the list would be sorted as before.

#### Dashboards app: Selected items in the feeds of a dashboard would be unselected after renaming that dashboard \[ID_23877\]

When you renamed a dashboard after selecting a number of items in its feeds, in some cases, all items selected in those feeds would incorrectly be unselected after the dashboard had been renamed.

#### Connection not closed when element with 'Accepted IP address' configuration was stopped \[ID_23882\]

If an element with a smart-serial connection was configured to only allow communication from certain IP addresses and this element was stopped, it could occur that the connection with those IP addresses was not properly closed.

#### Dashboards app: Indices data feed overwritten if same primary key was used \[ID_23884\]

If a dashboard contained a component using an indices data feed, and another indices data feed with the same primary key was added to a different component of the dashboard, it could occur that this second indices feed overwrote the indices for the first component.

#### Aggregation rules operating on views would generate an excessive amount of calls to SLDataMiner \[ID_23892\]

When an aggregation rule that operates recursively on a view was executed, in some cases, an excessive amount of calls from SLProtocol to SLDataMiner were generated.

#### Services app would not list all function definitions configured in newly activated functions file \[ID_23896\]

When you activated a new functions file for a particular protocol, in some cases, the Services app would not list all function definitions configured in that file.

#### DataMiner Cube: Memory leak when opening and closing dynamic list \[ID_23910\]

If a dynamic list component was opened and closed in DataMiner Cube, for instance in the Bookings app or the Services app, a memory leak could occur.

#### Problem with alarm properties after importing alarms \[ID_23918\]

When a DELT package containing alarms was imported, in some cases, alarm properties would incorrectly be added to alarms to which they did not belong.

#### Redundancy groups with multiple primary and multiple backup elements could go into an undefined state after switching \[ID_23924\]

In some cases, redundancy groups with multiple primary elements and multiple backup elements could incorrectly go into an undefined state after switching.

#### Problem with SLDataMiner when initializing redundancy groups at DataMiner startup \[ID_23943\]

During DataMiner startup, in some rare cases, an error could occur in SLDataMiner when initializing redundancy groups.

#### DataMiner Cube - Visual Overview: Problem with client-side filtering of dynamically positioned shapes \[ID_23946\]

When dynamically positioning shapes based on table data, in some cases, a row filter specified in a shape data field of type *Filter* would no longer be applied.

#### DataMiner Cube - Visual Overview: Problem when using a children shape in conjunction with a Service Overview Manager element \[ID_23960\]

In some cases, an exception could be thrown when using a children shape in conjunction with a Service Overview Manager element.

#### Problem with SLAnalytics when an element was removed \[ID_23965\]

In some rare cases, an error could occur in SLAnalytics when an element was removed.

#### Service & Resource Management: Icons defined in system functions would not be displayed in DataMiner Cube \[ID_23966\]

In some cases, icons defined in system functions would not be displayed in DataMiner Cube.

#### DataMiner Cube - Service templates: Incorrect combination values in 'Conditions' section of 'Edit child element' window \[ID_23979\]

When configuring conditions in the *Conditions* section of an *Edit child element* window, in some cases, the *Combination* selection boxes would contain incorrect values. Also, no value would be selected by default.

#### Table cells could no longer be set to 'invalid' & problem with SLElement when a user subscribed to a view table \[ID_23997\]

Due to a validation check problem, table cells could no longer be set to “invalid”.

Also, in some cases, an error could occur in SLElement when a user subscribed to a view table.

#### DataMiner packages: Problem when parsing UpdateContent.txt \[ID_24027\]

When a non-Latin character set was used in the UpdateContent.txt file of a DataMiner package, in some cases, an error could occur when that file was parsed.

#### Dashboards app: Problem when multiple view filters had been added to a parameter feed \[ID_24030\]

When multiple view filters had been added to a parameter feed using the entire element dataset, it could occur that only the last filter was taken into account.

#### DataMiner Cube - Trending: Problem when adding a parameter to a stopped element \[ID_24045\]

In some rare cases, an error could occur when you tried to add a parameter of a stopped element to a trend graph.

#### Problem with custom branding on landing page and tools page \[ID_24050\]

In some cases, it could occur that custom branding was not applied to a DataMiner Agent’s landing page and tools page.

#### Problem with SLPort when closing a serial or GPIB port \[ID_24056\]

In some rare cases, an error could occur in SLPort when a serial port or a GPIB port was closed.

#### DataMiner Cube - Trending: Problem with 'Enable trend logging (debug)' setting \[ID_24065\]

When the “Enable trend logging (debug)” setting was deactivated, in some cases, certain log data would incorrectly still be kept in memory.

#### DataMiner Cube: Server settings without value would incorrectly inherit the value set during the previous Cube session \[ID_24069\]

When a (new) server setting did not have a value, in some cases, instead of being assigned the default value it would incorrectly inherit the value that was assigned to it during the previous Cube session.

#### Problem during SLElement start-up \[ID_24075\]

When SLElement started up, in some cases, an error could occur when determining the amount of parameter threads to be created.

#### Scheduler: Problem when trying to save a scheduled task \[ID_24085\]

When a scheduled task was created with a report action that included a parameter with a filter containing a “\<“ character, it could occur that it was not possible to save that task.

#### Memory leak in SLElement after deleting DVE elements and/or virtual functions \[ID_24089\]

In some cases, it could occur that memory was not released properly in SLElement after DVE elements or virtual functions had been deleted.

#### Ticketing: Ticket fields containing concatenations would have their type set incorrectly \[ID_24092\]

When a ticket field contained a concatenation, in some cases, the type of that field would be set incorrectly.

#### Problem with SLNet when using protocol buffer serialization \[ID_24099\]

In some rare cases, a problem could occur in SLNet when protocol buffer serialization was being used.

#### Cassandra: Paged database query would incorrectly only return the first two pages \[ID_24101\]

In some cases, a paged database query against a Cassandra database would incorrectly only return the first two pages instead of the entire result set.

#### Parameters exported to multiple types of DVE elements would not be updated in all DVE elements \[ID_24103\]

When a parameter that was not directly linked to a DVE element via primary or foreign key was exported to multiple types of DVE elements sharing the same parent DVE element, it could occur that updates of that parameter did not get pushed to all the DVE elements to which it had been exported.

#### DataMiner Cube: Updates made to the LDAP settings would not immediately be applied \[ID_24110\]

When changes had been made to the LDAP settings, in some cases, those changes would not be applied immediately. From now on, any change made to the LDAP settings will be applied immediately.

#### Cassandra migration: Incorrect estimation of the amount of data to be migrated \[ID_24117\]

To be able to indicate the progress of a data migration operation, DataMiner first has to make an estimation of the amount of data to be migrated. Up to now, due to an incorrect calculation of logger table sizes, this estimation was not correct.

#### DataMiner Cube - Visual Overview: DCF connection lines between dynamically positioned shapes would disappear after zooming in and out \[ID_24119\]

In some cases, DCF connection lines between dynamically positioned shapes would disappear after zooming in and out.

#### Dashboards app: Problem when a parameter state component was linked to an element feed \[ID_24136\]

When the element filter of a parameter state component was changed, in some cases, that change would not get applied. When a parameter state component was linked to an element feed, only the initial filter would be applied. Any changed to the element feed would not be passed to the parameter state component.

#### Problem when changing a foreign key value in a table linked to a DVE element or a Virtual Function \[ID_24141\]

When a foreign key value was changed in a table linked to a DVE element or a Virtual Function, in some cases, only the primary key and foreign key value would be passed along with the update message. As a result, all other values would incorrectly be shown as “not initialized”.

#### Stopping an element with virtual functions included in an enhanced service would not correctly clear the alarms in the active alarms table \[ID_24154\]

When an element with virtual functions included in an enhanced service was stopped, although the alarms would disappear from the Alarm Console they would not be correctly cleared in the active alarms table of the enhanced service.

#### Performance issues in case of several subscriptions on single connection \[ID_24156\]

If several subscriptions were made on a single connection, there could be performance issues in Cube. This could for example be the case when service definitions were loaded.

#### Problem when simultaneously stopping an element with a direct view table and the element containing the source data \[ID_24169\]

In some cases, an error could occur when you simultaneously stopped an element with a direct view table and the element containing the source data of that direct view table.

#### DataMiner Cube - Cassandra: Problem when performing alarm queries \[ID_24177\]

On a DataMiner Agent with a Cassandra database but no Elastic database, in some cases, an exception could be thrown when performing an alarm query.

#### Problem when adding or deleting ObjectRefTreeElement objects in a Cassandra database \[ID_24188\]

In some cases, an error could be thrown when adding or deleting an ObjectRefTreeElement object in a Cassandra database.

#### DataMiner Cube: Problem when closing card \[ID_24196\]

In some rare cases, a problem could occur in DataMiner Cube when a card was closed.

#### Credential library: Problem with passwords having been encrypted multiple times \[ID_24201\]

In the credential library, in some cases, passwords would incorrectly be encrypted multiple times.

Also, in some cases, an exception could be thrown when you tried to edit an existing library credential.

#### Service & Resource Management: When a service definition was updated, the old version would not be removed \[ID_24208\]

When a service definition was updated, in some cases, the old version of that service definition would incorrectly not be removed from the *servicedeftrace* database table.

#### DataMiner Cube - System Center: A backup including the Indexing database could incorrectly be started without specifying a backup path \[ID_24217\]

In the *Backup* section of *System Center*, users would incorrectly be able to start a backup that included the Indexing database even when no *Indexing Engine backup path* had been specified. As a result, the backup operation would fail immediately.

From now on, the *Execute backup* button will be disabled when no *Indexing Engine backup path* is specified.

#### DataMiner Cube - System Center: Agent list would incorrectly be expanded after having chosen not to upgrade a number of specific agents \[ID_24223\]

When, in the *Agents* section of *System Center*, you click *Upgrade*, you can choose whether to upgrade either all agents in the cluster or a number of specific agents. If you choose the latter, and click “Yes” in the confirmation box, the list of available agents will be expanded, allowing you to select the agents to be upgraded.

However, up to now, the list of available agents would also incorrectly be expanded when you clicked “No” in the confirmation box. This will no longer occur.

#### Service & Resource Management: Resources in service of child booking not updated after booking resources were updated \[ID_24224\]

When an SRM script updated the resources for a child booking but it only passed the main booking to the *AddOrUpdateReservationInstances* call, it could occur that the resources in the service of the child booking were not updated.

#### DataMiner Cube: Version conflict error after logging in with an incorrect Administrator password \[ID_24229\]

When you tried to log in to DataMiner Cube with an incorrect Administrator password, in some cases, a version conflict error could be thrown.

#### DataMiner Cube: Locally saved document locked until DMA restart \[ID_24231\]

When a document was saved from the Documents app in Cube to a local drive, it could occur that this file was locked by DataMiner until the DMA was restarted.

#### Service & Resource Management: Alarm icons not displayed in ListView component \[ID_24232\]

In some cases, if the column configuration for a ListView component contained alarm icons, it could occur that the alarm icons were not displayed when the configuration was loaded.

#### Jobs app: Problem when filtering on a field of the default job section \[ID_24236\]

When creating a filter on a field of the default job section, in some cases, an incorrect field ID would be used.

#### Issues when deleting booking instances \[ID_24244\]

When a booking instance in "Ongoing" state was deleted, it could occur that the deletion failed. In addition, if a booking instances tree was deleted by passing the nodes in the tree to the helper call, it could occur that the response contained too many bookings.

#### Element migration failure because of file with same name \[ID_24249\]

When an element was migrated, it could occur that an element data file could not be created because a file with the same name already existed, causing the migration to fail. Now a suffix will be added to the file name to ensure that the file can be created.

#### DataMiner Cube - Failover: Progress message no longer displayed during a switch-over \[ID_24250\]

During a switch-over, in some cases, users would no longer receive a “\<DMA> is switching to \<DMA>” message.

Also, the term “System Display” has been replaced by “Client Interface” in *Failover status* windows.

#### DataMiner Cube - Alarm Console: Alarm filters using session variables would no longer be updated correctly \[ID_24255\]

In some cases, alarm filters that contained session variables would no longer be updated correctly when one of those variables had changed.

#### Dashboards app: Incorrect message would appear when an interactive Automation script went into timeout \[ID_24258\]

When an interactive Automation script running inside the Dashboards app went into timeout, in some cases, a success message would incorrectly be displayed instead of an error message.

#### DataMiner Cube - Profiles app: Protocol version not set to 'Production' when linking a parameter \[ID_24260\]

When, in the Profiles app, you dragged an element using the production version of a protocol onto the parameter list and linked a protocol parameter to a profile parameter, in some cases, the protocol version would incorrectly not be set to “Production”.

#### DataMiner Cube - Protocols & Templates: Problem when opening information template \[ID_24267\]

In some cases, an exception could be thrown when an information template was opened.

#### Trending: Incorrect trend data retrieved for view column parameters \[ID_24268\]

In some cases, incorrect trend data would be retrieved for view column parameters.

#### DataMiner Cube - Visual Overview: Missing options in trending context menu \[ID_24271\]

In some cases, it could occur that the context menu for a trend component in Visual Overview did not show all the available options.

#### Problem with SLDataMiner when processing a service alarm while updating a service \[ID_24274\]

In some rare cases, an error could occur in SLDataMiner when it is was simultaneously updating a service and processing a new alarm linked to a service.

#### DataMiner Cube - Profiles app: When linking a profile parameter to a protocol parameter only Read/Write parameters would be listed \[ID_24280\]

When, in the Profiles app, you linked a profile parameter to a protocol parameter, the parameter selection box would incorrectly only list Read/Write parameters.

Also, it would incorrectly only be possible to link parameters of category “Configuration” to protocol parameters.

#### Datetime value in interactive Automation script not updated correctly \[ID_24284\]

When a datetime value in an interactive Automation script was changed by the script itself, it could occur that the value was not updated correctly.

#### Dashboards app / Jobs app: Problem opening the user menu \[ID_24285\]

In the Dashboards app and the Jobs app, in some cases, it would not be possible to open the user menu.

#### Problem in SLElement when adding comment while clearing timeout alarm \[ID_24286\]

When a timeout alarm was cleared manually and the user set the comment when clearing the alarm to the IP address of the connection, a problem could occur in SLElement.

#### Jobs app: Job end times displayed incorrectly in timeline \[ID_24287\]

In the jobs timeline, in some cases, job end times would be displayed incorrectly.

#### Problem when filtering view tables \[ID_24288\]

When a view table was filtered, in some cases, the table would incorrectly be empty or show incorrect data.

#### Failover - MySQL: Alarm cleaning process would not be able to correctly separate active from non-active alarms \[ID_24306\]

When the number of alarms stored in the database reaches a certain threshold, the oldest, non-active alarms are automatically removed from the system. However, on a Failover system with MySQL databases, in some cases, the alarm cleaning process would not be able to correctly separate active from non-active alarms.

#### Problem when DVE creation was being disabled while updating a table with an ';element' column option \[ID_24309\]

In some rare cases, an error could occur when a table with an “;element” column option was being updated while a request was being processed to disable the DVE creation on that element.

#### Problem with a FullFilter on a direct view that retrieved data from elements with different protocols \[ID_24313\]

When a FullFilter was used on a direct view that retrieved data from elements with different protocols, in some cases, no results would be returned.

#### DataMiner Cube - Data Display: Problem when displaying a tabbed table \[ID_24317\]

In some cases, an exception could be thrown when a tabbed table was displayed on a DATA page.

#### DataMiner Cube: Masked alarms included in 'Unassigned active alarms' or 'My active alarms' tab page \[ID_24319\]

When an "Unassigned active alarms" or "My active alarms" tab page was added in the Alarm Console, it could occur that this tab page also included masked alarms.

#### Reports & Dashboards: Empty tables in legacy dashboards \[ID_24320\]

On legacy dashboard, in some cases, tables containing data would incorrectly be displayed as empty.

#### DataMiner Cube: Not possible to connect to another DMA after starting Cube with a 'host=' argument \[ID_24322\]

When DataMiner Cube was started with a “host=” argument, in some cases, the user would not be able to connect to another DataMiner Agent.

#### HTML5 apps: Selecting the month value in a datetime control would incorrectly clear that value \[ID_24323\]

When you selected the month value in a datetime control, in some cases, that value would incorrectly be cleared.

#### Exception when starting up DMAs in cluster with empty Elastic database \[ID_24325\]

If several DMAs in a cluster were started at the same time and these DMAs has an empty Elastic database, an exception could be thrown.

#### DataMiner Cube - Visual Overview: Problem when using dynamic zoom while DCF connection properties were being displayed \[ID_24326\]

In some rare cases, an exception could be thrown when you tried to use the dynamic zoom while DCP connection properties were being displayed.

#### Automation: Dialog box from an interactive Automation script started in Dashboards would incorrectly appear in Cube \[ID_24328\]

In some cases, a dialog box from an interactive Automation script started in Dashboards would incorrectly appear inside a Cube session.

#### Reports & Dashboards: Problem with status query in legacy Reporter app \[ID_24332\]

When you used the legacy Reporter app to generate a report containing a status query, in some cases, a “response buffer limit exceeded” error message would appear, especially when the report contained a large amount of data.

#### Alarm Console: Hyperlinks missing in right-click menu after element restart or DataMiner restart \[ID_24335\]

In the Alarm Console, in some cases, the alarm hyperlinks would disappear from the right-click menu after an element restart or a DataMiner restart.

#### Alarm templates: Problem when changing the condition of a conditionally monitored parameter \[ID_24338\]

When, in an alarm template, a conditionally monitored parameter had its condition changed, in some cases, alarms associated with that parameter would incorrectly be cleared using the current time instead of the time at which the condition was changed.

Also, when such a condition was changed, the alarm level update would be stored in the database with an incorrect time stamp, leading to issues with trend graphs.

#### Problem when importing DELT packages containing alarms \[ID_24340\]

When a DELT package containing alarms was imported on a system running Cassandra, in some cases, the root time and root creation time of the alarms could be incorrect.

#### Service & Resource Management: Function DVEs not removed after resource swap \[ID_24342\]

If resources of a running booking were swapped with other resources, it could occur that the function DVEs that were no longer in use were not disabled until the maximum number of functions was reached.

#### DataMiner Cube: Not possible to navigate to DataMiner object from embedded Chromium browser \[ID_24343\]

When an embedded Chromium browser was used in DataMiner Cube, it could occur that it was not possible to navigate to a DataMiner object from the embedded browser.

#### DataMiner Cube: CPE card headers not showing the correct alarm color \[ID_24345\]

In some cases, the header of a CPE card did not show the correct alarm color. Instead, it was set to gray (Initial).

#### Element connections saved incorrectly \[ID_24348\]

If an element had multiple serial, smart-serial or HTTP port connections, it could occur that the connection type of the first port was applied to all other serial, smart-serial or HTTP ports as well.

#### DataMiner Cube - Automation: Variables not linked to an input parameter would not be displayed \[ID_24358\]

In Automation, in some cases, referenced variables would incorrectly not be displayed if they were not linked to a script input parameter.

#### Service & Resource Management: Exception when saving multiple profile instances or profile definitions with empty field \[ID_24359\]

When multiple profile instances or profile definitions containing an empty field were saved at the same time, an exception could be thrown.

#### DataMiner Cube: Pages loading slowly because of unnecessary view table data refresh \[ID_24360\]

In some cases, it could occur that view table data were refreshed even when this was not necessary, which could cause data pages or visual overview pages to load very slowly in Cube.

#### DataMiner Cube - Correlation: Problem with 'Send email' actions in Correlation rules \[ID_24361\]

When triggered, in some cases, “Send email” actions in Correlation rules would not send an email when the recipient was a DataMiner user.

#### DataMiner Cube: Problem when resizing the Alarm Console \[ID_24362\]

When you made the Alarm Console smaller and then restored it to its original size, in some cases, the entire console would become blank.

#### DataMiner Cube: Not possible to export debug information \[ID_24365\]

In some cases, it could occur that it was not possible to export debug information in the Logging app.

#### SNMP forwarding with multiple alarm filters not working correctly \[ID_24368\]

If multiple alarm filters were combined for the forwarding of SNMP notifications, it could occur that only the first filter was applied.

#### DataMiner Cube: Problem when changing the protocol as well as the trend or alarm template of an element \[ID_24371\]

In some cases, an exception could be thrown when, during an element edit, you changed the protocol or protocol version as well as the trend or alarm template.

#### Service & Resource Management - Bookings app: Bookings displayed on the bookings timeline would not be listed in the bookings list \[ID_24374\]

In the Bookings app, in some cases, bookings displayed on the bookings timeline would not be listed in the bookings list.

#### DataMiner Cube: Problem when displaying dialog box while window/scroll bar thumb is dragged \[ID_24383\]

In the XBAP version of DataMiner Cube, if a dialog box was displayed while a user was dragging a window or a scroll bar thumb, a problem could cause Cube to freeze.

#### Booking definition saved even though no instance could be planned \[ID_24397\]

In some cases, it could occur that a booking definition was saved even though it was not possible to plan a booking instance.

#### Addition/removal booking events of existing booking not implemented correctly \[ID_24398\]

When booking events were added to or removed from an existing booking that still needed to start, it could occur that this change was not executed correctly.

#### Service & Resource Management: Exceptions when using a MySQL local database \[ID_24401\]

On systems with a MySQL local database, if a booking was saved that overrides a property on one of its resources, an exception could be thrown. In addition, an exception could be thrown during the initialization of the Resource Manager.

#### DataMiner Maps: Problem with user-defined links in pop-up balloons when using the Chromium web engine \[ID_24402\]

When DataMiner maps were being displayed on a Visio page using a Chromium web engine, in some cases, the user-defined links in pop-up balloons would not function correctly.

#### DataMiner Cube - Visual Overview: Incorrect trend legend when graph was opened from table parameter in Visio \[ID_24405\]

When you opened a trend graph from a table parameter control in Visio, it could occur that the legend of the trend graph only allowed a limited parameter selection.

#### Landing page displayed link to Ticketing even without Ticketing license \[ID_24412\]

If a DMA was not licensed for Ticketing, it could occur that the DataMiner landing page nevertheless displayed a link to the Ticketing app.

#### DataMiner Cube - Visual Overview: Column configuration of ListView component not updated \[ID_24422\]

In some rare cases, it could occur that the column configuration of a *ListView* component was not saved or retrieved correctly.

#### DataMiner Cube: Editing an SNMPv3 element created prior to DataMiner 9.6.12 would cause the authentication type to be reset to the default type \[ID_24423\]

When you edited an SNMPv3 element that was created prior to DataMiner 9.6.12, in some cases, the authentication type would incorrectly be reset to the default type (i.e. SHA512).

#### Tables would show incorrect data after being updated \[ID_24425\]

When protocol buffer serialization was enabled, in some cases, tables would show incorrect data after being updated.

#### Dashboards: Problem with duplicate trend graphs in line chart component \[ID_24427\]

When, in a line chart component, you selected a second parameter, in some cases, each trend graph would incorrectly be displayed twice.

#### Service & Resource Management: ReservationInstances did incorrectly not have their status set to 'interrupted' \[ID_24434\]

In some rare cases, ReservationInstances that should have had their status set to “interrupted”, had their status set to an incorrect value.

#### Cassandra: Problem during upgrade would cause duplicate entries being added to the activealarms table \[ID_24436\]

In some cases, an error could occur when executing the *CassandraActiveAlarmsRootOnlyUpgrade* upgrade action. As a result of this error, duplicate entries would be added to the activealarms table.

#### Problem with SLNet when the connections among the different DataMiner Agents were being initiated \[ID_24437\]

In some rare cases, an error could occur in SLNet when the DataMiner Agents in the DataMiner System were initiating the connections among each other.

#### DataMiner landing page: Broken links to other pages \[ID_24439\]

On the DataMiner landing page, in some cases, the following links would be broken:

- Link to the <https://www.skyline.be> website (under “Skyline Communications”)
- Link to /licenses.asp
- Link to /systemdisplay.htm

#### DataMiner Cube: Problem when searching paged tables \[ID_24440\]

When you entered a string enclosed in double quotes ("") in the search box of a paged table, in some cases, an incorrect filter would be sent to the DataMiner Agent.

Also, it would not be possible to search for exception values or discreet values.

#### Problem with SLElement when updating the source data of a direct view table belonging to an element with debug log level 2 \[ID_24449\]

In some cases, an error could occur in SLElement when updating the source data of a direct view table belonging to an element of which the debug log level was set to 2.

#### DataMiner Cube - Surveyor: Views without sub-views would incorrectly have an expand arrow \[ID_24465\]

In the Surveyor, in some cases, views without sub-views would incorrectly have an expand arrow.

#### DataMiner Cube - Services: Some rows would be missing from tables partly included in a service \[ID_24471\]

In some cases, rows could be missing from tables that were partly included in a service.

#### Backup aborted if package could not be copied to network drive \[ID_24482\]

If a DataMiner backup package could not be copied to a network drive, it could occur that the backup was aborted. The new backup package on the DMA would still be marked as a temporary package, which was deleted when the next backup started.

#### DataMiner Cube - Data Display: Memory leak after sorting and/or filtering tables \[ID_24492\]

In some cases, DataMiner Cube could leak memory after sorting and/or filtering tables displayed on DATA pages.

#### Dashboards app: Element tree of a multiple-parameter feed was sorted incorrectly \[ID_24495\]

In some cases, the element tree of a multiple-parameter feed would not be sorted correctly. From now on, this tree will be sorted correctly, even when multiple filters are applied.

#### DataMiner Cube: Miscellaneous Alarm Console issues \[ID_24508\]

When a sliding window tab in the Alarm Console included cleared alarms, it could occur that the severity duration for these alarms was not indicated correctly. In addition, when you opened the alarm card of a cleared alarm, it could occur that no severity duration was shown for this alarm. Finally, when you opened the alarm card for an alarm that was not the top alarm in an alarm tree, only that alarm was shown on the card. Now the entire alarm tree will be shown in that case.

#### Dashboards app: Problem with unintentional move operations in Chrome \[ID_24525\]

When working with the Dashboards app in Chrome, in some cases, a mouse click could unintentionally cause a dashboard component to be moved to another location.

#### Dashboards app: Parameters in the URL would not correctly be loaded in the multiple parameter feed \[ID_24536\]

When you opened a dashboard with a multiple parameter feed using a URL that contained the parameters to be loaded in that feed, in some cases, the parameters in the URL would not correctly be loaded into the multiple parameter feed.

#### HTML5 apps: Selection box values containing backslash characters displayed incorrectly in interactive Automation scripts \[ID_24541\]

When an interactive Automation script was run from within an HTML5 app, in some cases, selection box values containing “\\” characters could be displayed incorrectly.

#### Failover: Problem with Indexing database after a Failover switch \[ID_23945\]\[ID_24562\]

When an Indexing database was installed on a pair of DataMiner Agents in a Failover setup, in some cases, the Indexing database could no longer be reached after a Failover switch.

#### Problem when opening an HTML5 app \[ID_24565\]

When you opened an HTML5 app (e.g. Monitoring & Control, Dashboards, etc.), in some cases, the app would not load due to a missing internal component.

#### Service & Resource Management: Problem during Resource Manager initializing \[ID_24604\]

In some cases, the Resource Manager module could fail to initialize when protocol buffer serialization was enabled.

#### Service & Resource Management: Problem when retrieving a ReservationInstance immediately after a property update \[ID_24622\]

When a ReservationInstance was retrieved immediately after its properties had been updated, in some cases, the properties of the retrieved ReservationInstance would incorrectly still have their old values.

#### DataMiner Cube - Alarm console: Dragging an element to the Alarm Console incorrectly showed all alarms on the DataMiner Agent \[ID_24655\]

When you dragged an element onto the Alarm Console, in some cases, the alarm tab would incorrectly list all alarms on the DataMiner Agent instead of only the alarms of the element in question.

## Changes in DataMiner 10.0.2 CU1

### CU1 fixes

#### Problem when logging in to a DataMiner mobile app \[ID_24333\]

In some cases, the following exception could be thrown when logging in to any DataMiner mobile app:

```txt
Failed to setup a connection with the DataMiner Agent: Method not found:'SLLoggerUtil.ILogger SLLoggerUtil.LoggerUtil.GetLogger(SLLoggerUtil.LoggerCategoryUtil.ILoggerCategory)'.
```

#### Problem with SLProtocol when calling 'NT_LOAD_TABLE' \[ID_24780\]

In some cases, an error could occur in SLProtocol when calling the NotifyProtocol method “NT_LOAD_TABLE”.

#### DataMiner Cube failed to start \[ID_24796\]

On a clean Windows system, in some cases, DataMiner Cube would fail to start, especially when using a new user account.

#### Problem when trying to back up an Elastic database using its public IP address \[ID_24810\]

When you tried to back up an Elastic database (which is used by the DataMiner Indexing engine) using its public IP address, in some cases, a message would incorrectly appear, saying that no Elastic database could be found on the machine in question.

## Changes in DataMiner 10.0.2 CU2

### CU2 enhancements

#### Service & Resource Management: Enhanced performance when deleting service definitions \[ID_24860\]

Due to a number of enhancements, overall performance has increased when deleting service definitions, especially on systems with a large number of booking instances.

### CU2 fixes

#### DataMiner Cube - Services app: Service definition not loaded correctly when Services app is opened \[ID_24735\]

When you opened the Services app, it could occur that one of the service definitions was not loaded correctly. When you selected it, it was tagged as “\[modified\]” and its connection lines were lost.

#### DataMiner - Services app: Problem when saving service definitions \[ID_24849\]

In some cases, an error could occur when you tried to save a service definition.

#### Migrating booking data to DataMiner Indexing failed when service definition properties had null values \[ID_24880\]

If service definition properties had null values, it could occur that migrating booking data to DataMiner Indexing failed.

#### Failover - Service & Resource management: Problem when migrating booking data to DataMiner Indexing after performing a Failover switch twice \[ID_24883\]

When you started a booking data migration to DataMiner Indexing on a Failover Agent that had previously been switched twice, in some cases, the migration operation would never get marked as completed.
