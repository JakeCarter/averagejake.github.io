# Starbucks Tech Talk

## Destiny Modifiers Demo
- App: __663f6e26__
- Framework: __3bf0a9ba__
- Apply APIKey Stash

### Demo 0: Demo the App
- Show off framework
	- Separation of Internals vs Facade
- Show off working app

### Demo 1: Add Swift to App
- Add Swift Detail View

Filename:
	
	ModifierDetailViewController
	
- Point out Bridging header prompt
- Show changes Xcode made to project file.
- Circle back to stubbed in detail view controller in Storyboard
- Assign custom class
- Create `push` segue & fix Auto Layout
- Interface partially setup in IB and part in code
- Create outlet for `titleLabel`
- Create property for `descriptionLabel`

Swift Implementation:
	
	private let descriptionLabel: UILabel
	
 - Non-optional, NOT setup in IB
 - Can use closure to initialize stored properties

Initialization Closure:
	
	= {
			let label = UILabel()
			label.translatesAutoresizingMaskIntoConstraints = false
			label.numberOfLines = 0
		
			return label
	}()
	
- Add `descriptionLabel` and Auto Layout

Add to `viewDidLoad()`:
	
	view.addSubview(descriptionLabel)
	descriptionLabel.topAnchor.constraint(equalTo: titleLabel.bottomAnchor, constant: 8.0).isActive = true
	descriptionLabel.rightAnchor.constraint(equalTo: view.layoutMarginsGuide.rightAnchor, constant: 0.0).isActive = true
	descriptionLabel.leftAnchor.constraint(equalTo: view.layoutMarginsGuide.leftAnchor, constant: 0.0).isActive = true
	
Setup title:
	
	navigationItem.title = "Details"
	
- Need property for modifier

Modifier Property:
	
	var modifier: DNIItem?
	
- Must be optional 
- Display `modifier` in UI

Add to bottom of `viewDidLoad()`:
	
	if let modifier = modifier {
		titleLabel.text = modifier.title
		descriptionLabel.text = modifier.itemDescription
	}

Property Observer (Bonus if time):
	
	var modifier: DNIItem? {
		didSet {
			titleLabel?.text = modifier?.title
			descriptionLabel.text = modifier?.itemDescription
		}
	}
	
Implement `-[NightfallModifiersTableViewController prepareForSegue:]`:
	
	- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender {
		if ([segue.destinationViewController isKindOfClass:[ModifierDetailViewController class]]) {
			NSAssert([sender isKindOfClass:[UITableViewCell class]], @"sender is expected to be of type UITableViewCell");
			UITableViewCell *selectedCell = (UITableViewCell *)sender;
			NSIndexPath *selectedIndexPath = [self.tableView indexPathForCell:selectedCell];
			DNIItem *modifier = self.modifiers[selectedIndexPath.row];
			
			ModifierDetailViewController *detailViewController = (ModifierDetailViewController *)segue.destinationViewController;
			detailViewController.navigationItem.title = @"Hello, Swift Bridge!";
			detailViewController.modifier = modifier;
		}
	}
	
Import _Xcode-generated header_:
	
	#import "DestinyNightfallModifiers-Swift.h"
	
- Kind of like an umbrella header for our Swift code
- Run app; Show detail screen

_Should now be at App: __b8d170ad__ Framework: 3bf0a9ba_

### Demo 2: Swiftify Framework
- Show generated header for `DNIClient`
- Add `NS_ASSUME_NONNULL` wrappers

Begin:
	
	NS_ASSUME_NONNULL_BEGIN
	

End:
	
	NS_ASSUME_NONNULL_END
	

- Show generated header for `DNIClient` again
- Point out non-optional `nightfallModifiers` and `error` closure arguments
- Mark each as `_Nullable`

Nullable Attribute:
	
	_Nullable
	
- `nightfallModifiers` is an array of what?
- Add generic type annotation

Generic array annotation:
	
	<DNIItem *>
	
- Copy definition to .m file to fix build error
- Show generated header for `DNIItem`
- Initializer returns an optional and it's arguments are optional and shouldn't be
- Add `NS_ASSUME_NONNULL` wrappers

Begin:
	
	NS_ASSUME_NONNULL_BEGIN
	

End:
	
	NS_ASSUME_NONNULL_END
	

- Show `DNIFetcher`; No need for details.
- Same as `DNIClient`

Reimplement with:
	
	NS_ASSUME_NONNULL_BEGIN
	
	@interface DNIFetcher : NSObject
	
	- (void)fetchAdvisorsJSONObjectWithCompletionHandler:(void (^)(id _Nullable jsonObject, NSError * _Nullable error))completionHandler;
	
	@end
	
	NS_ASSUME_NONNULL_END
	
- Point out `NS_ASSUME_NONNULL` wrappers
- Point out `_Nullable` annotations

_Should now be at App: b8d170ad Framework: __389432f4___

### Demo 3: Add Swift to Framework
- Add new Swift File

Filename:
	
	DNIClient-Additions.swift
	
- Show changes Xcode made to project file.

Stub in extension:
	
	extension DNIClient {
		public typealias DNIModifierFetchCompletionHandler = ([DNIItem]?, Error?) -> Void
		typealias InfoDict = [String : Any]
		
		public func heroicStrikeModifiers(completionHandler: @escaping DNIModifierFetchCompletionHandler) {
		
		}
	}
	
- Show how `DNIClient` gets the JSON from the `fetcher`
- Point out that the `fetcher` property is hidden from subclasses and extensions because it is in the class extension
- Move `fetcher` property to .h

Property comment:
	
	// Internal - Only here for Swfit Bridging
	
Forward class:
	
	@class DNIFetcher;
	
- Swift still cannot see `fetcher` property or `DNIFetcher` type.
- Add to umbrella header

Private Import:
	
	#import "DNIFetcher.h"
	

- Show build error; Can't be private;
- Make `DNIFetcher` public
- Update import in umbrella header

Public Import:
	
	#import <DestinyNightfallInfo/DNIFetcher.h>
	
- Verify `fetcher` property is useable now
- Implement `heroicStrikeModifiers()`

`heroicStrikeModifiers()`:
	
	self.fetcher.fetchAdvisorsJSONObject { (advisorsInfo, error) in
		guard let advisorsInfo = advisorsInfo as? InfoDict,
		let responseInfo = advisorsInfo["Response"] as? InfoDict,
		let dataInfo = responseInfo["data"] as? InfoDict,
		let activitiesInfo = dataInfo["activities"] as? InfoDict,
		let heroicInfo = activitiesInfo["heroicstrike"] as? InfoDict,
		let extendedInfo = heroicInfo["extended"] as? InfoDict,
		let skullCategories = extendedInfo["skullCategories"] as? [Any],
		let skullCategory = skullCategories[0] as? InfoDict,
		let skullInfos = skullCategory["skulls"] as? [InfoDict] else {
			completionHandler(nil, error)
			return
		}
		
		let modifiers = skullInfos.map { (infoDict) -> DNIItem in
		let title = infoDict["displayName"] as! String
		let itemDescription = infoDict["description"] as! String
		
		let item = DNIItem(title: title, itemDescription: itemDescription)
			return item
		}
		
		completionHandler(modifiers, nil)
	}
	
- Explain `guard` statement
- Explain map

_Should now be at App: b8d170ad Framework: __89959627___

### Demo 4: Use Framework Swift
- Need to use Swift code from Framework but only importing umbrella header; Switch to @import for module

Module Import:
	
	@import DestinyNightfallInfo;
	

- Uncomment alert controller code in `NightfallModifiersTableViewController`

Implement `_fetchHeroic`:
	
	self.title = @"Heroic";
	if (self.client == nil) {
		self.client = [[DNIClient alloc] init];
	}
	
	[self.client heroicStrikeModifiersWithCompletionHandler:^(NSArray<DNIItem *> * _Nullable heroicModifiers, NSError * _Nullable error) {
		if (heroicModifiers == nil) {
			NSLog(@"Error fetching Nightfall Modifiers: %@", error.localizedDescription);
			return;
		}
		
		self.modifiers = heroicModifiers;
		dispatch_async(dispatch_get_main_queue(), ^{
			[self.tableView reloadData];
		});
	}];
	
_Should now be at App: __3708163c__ Framework: 89959627_
