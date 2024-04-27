---
layout: post
title:
author:
date: 2023-01-01
categories:
---

<span class="lead">Lorem ipsum dolor sit amet</span>, consectetur adipiscing elit.

~~~ javascript
const TableFlattener = (function () {
	/////// PRIVATE PROPERTIES ///////
	const DICT_FIELDS = [
		'element',
		'column_label',
		'internal_type',
		'attributes',
		'reference',
		'choice',
		'max_length'
	];

	const CHOICE_FIELDS = [
		'element',
		'value',
		'label',
		'sequence',
		'dependent_value',
		'inactive'
	];

	const relationTransformersByTable = {
		question_answer: QuestionAnswerTransformer,
		asmt_assessment_instance_question: AssessmentInstanceQuestionTransformer
	};

	/////// RETURN OBJECT ///////
	return {
		clone: clone,
		sync: sync
	};

	/////// PUBLIC METHODS ///////

	// Clone database structure from origin table to destination
	// table, including columns from relation table
	function clone (origin, relation, destination, query) {
		if (!/devprogleasing/.test(gs.getProperty('instance_name'))) {
			throw new Error('Database structure cloning is prohibited outside the `devprogleasing` instance.');
		}

		query = query || '';
		const gq = new global.GlideQuery.parse(origin, query);
		const relationTransformer = relationTransformersByTable[relation](origin, query);

		/////// DICTIONARIES ///////
		let destDictsLookup = getDictionaryList(destination) // Not const because it will be refreshed
			.reduce(global.GQX.toObject('element', false), {});

		const origDicts = getDictionaryList(origin)
			.filter(dict => gq.whereNotNull(dict.element).count()) // Skip empty fields -- *** NESTED QUERY ***

		origDicts
			.map(dict => {
				dict.element = `u_dict_${dict.element}`; // Avoid name collisions
				if (dict.choice === 0) dict.choice = null; // Silly GlideQuery returns 0 because integer
				return dict;
			})
			.concat(relationTransformer?.getDictionaries() || []) // Get fields from relation table
			.filter(dict => !areSame(dict, destDictsLookup[dict.element], DICT_FIELDS)) // Skip unchanged fields
			.map(dict => {
				delete dict.sys_id;

				let destDict = destDictsLookup[dict.element];
				if (destDict) dict.sys_id = destDict.sys_id;

				if (dict.sys_id) delete dict.column_label; // Set only on insert (might've been changed on purpose)

				dict.name = destination;
				dict.active = true;
				dict.read_only = true;

				return dict;
			})
			.forEach(dict => {
				// Insert or update destination dictionary
				new global.GlideQuery('sys_dictionary')
					.insertOrUpdate(dict);
			});

		// Refresh destination dictionaries since they may have changed
		destDictsLookup = getDictionaryList(destination)
			.reduce(global.GQX.toObject('element', false), {});

		/////// CHOICES ///////
		const destFields = Object.keys(destDictsLookup)
		const origFields = origDicts
			.map(global.GQ.get('element'))
			.reduce(global.GQX.toArray, []);

		const destChoicesLookup = getChoiceList(destination, destFields)
			.reduce(global.GQX.toObject('key', false), {});

		getChoiceList(origin, origFields)
			.map(choice => {
				choice.element = `u_dict_${choice.element}`; // Avoid name collisions 
				return choice;
			})
			.concat(relationTransformer?.getChoices() || []) // Get choices from relation table
			.filter(choice => Object.keys(destDictsLookup).includes(choice.element)) // Skip fields not in destination
			.filter(choice => !areSame(choice, destChoicesLookup[choice.key], CHOICE_FIELDS)) // Skip unchanged choices
			.map(choice => {
				delete choice.sys_id;

				let destChoice = destChoicesLookup[choice.key];
				if (destChoice) choice.sys_id = destChoice.sys_id;

				delete choice.key;
				choice.name = destination;

				return choice;
			})
			.forEach(choice => {
				// Insert or update destination choice
				new global.GlideQuery('sys_choice')
					.insertOrUpdate(choice);
			});
	}

	// Sync data from origin table to destination
	// table, including variable values
	function sync (origin, relation, destination, query) {
		query = query || '';

		const destFieldNames = getAllFieldNames(destination);
		const origFieldNames = getAllFieldNames(origin)
			.filter(fieldName => destFieldNames.includes(`u_dict_${fieldName}`)); // Skip fields not in destination

		const relationTransformer = relationTransformersByTable[relation](origin, query, destFieldNames);

		const records = global.GlideQuery.parse(origin, query)
			.select(origFieldNames)
			.reduce(global.GQX.toArray, [])
			.map(prefix('u_dict')) // Avoid name collisions
			.map(record => ({ ...record, ...relationTransformer.getValues(record.u_dict_sys_id) })) // Add values from relation
			.map(record => {
				new global.GlideQuery(destination)
					.where('u_dict_sys_id', record.u_dict_sys_id)
					.selectOne()
					.ifPresent((existingRecord) => {
						record.sys_id = existingRecord.sys_id
					});
				return record;
			});

		records.forEach(record => {
			new global.GlideQuery(destination)
				.insertOrUpdate(record);
		});

		// Delete destination records if origin record no longer exists
		new global.GlideQuery(destination)
			.where('u_dict_sys_id', 'NOT IN', records.map(global.GQ.get('u_dict_sys_id')))
			.deleteMultiple();
	}


	/////// PRIVATE METHODS ///////
	function prefix (prefix) {
		// Function currying
		return function (record) {
			for (let fieldName in record) {
				let prefixedFieldName = `${prefix}_${fieldName}`;
				record[prefixedFieldName] = record[fieldName];
				delete record[fieldName];
			}
			return record;
		};
	}

	function getDictionaryList (table) {
		let tables = getTableAndAllSupersList(table)
		
		return new global.GlideQuery('sys_dictionary')
			.where('name', 'IN', tables)
			.where('element', '!=', 'sys_id') // Skip for now, see below
			.where('active', true)
			.whereNotNull('element') // exclude, e.g., collections
			.select(DICT_FIELDS)
			.reduce(global.GQX.toArray, [])
			.concat(
				// There is a sys_id field for every table but we only want
				// one, so we skipped it before and add it separately here
				new global.GlideQuery('sys_dictionary')
					.where('name', table)
					.where('element', 'sys_id')
					.selectOne(DICT_FIELDS)
					.get() // Throws error if not found, but should be there
			);
	}

	function getChoiceList (table, fields) {
		return new global.GlideQuery('sys_choice')
			.where('name', table) // Naively assumes choices are never inherited
			.where('element', 'IN', fields)
			.select(CHOICE_FIELDS)
			.map(choice => {
				choice.key = choice.element + ' ' + choice.value;
				return choice;
			})
			.reduce(global.GQX.toArray, []);
	}

	function areSame (a, b, propertiesToCompare) {
		if (typeof a === 'undefined') return false;
		if (typeof b === 'undefined') return false;

		return propertiesToCompare.every(prop => a[prop] === b[prop]);
	}

	function getTableAndAllSupersList (tableName, resultArray) {
		if (typeof resultArray === 'undefined') resultArray = [tableName];
		else resultArray.push(tableName);

		const table = new global.GlideQuery('sys_db_object')
			.where('name', tableName)
			.whereNotNull('super_class.name')
			.selectOne('super_class.name')
			.orElse(false);

		if (!table) return resultArray;
		else return getTableAndAllSupersList(table.super_class.name, resultArray);
	}

	function getAllFieldNames (table) {
		const schema = global.Schema.of(table, ['*']);
		return Object.keys(schema[table]);
	}
})();
~~~

Cras mollis risus nec nisl vulputate lobortis.{% include endmark.html %}
