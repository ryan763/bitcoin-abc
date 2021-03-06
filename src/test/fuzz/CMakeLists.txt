# Fuzzer test harness
add_custom_target(bitcoin-fuzzers)

define_property(GLOBAL
	PROPERTY FUZZ_TARGETS
	BRIEF_DOCS "List of fuzz targets"
	FULL_DOCS "A list of the fuzz targets"
)
set_property(GLOBAL APPEND PROPERTY FUZZ_TARGETS bitcoin-fuzzers)

include(InstallationHelper)
macro(add_fuzz_target TARGET EXE_NAME)
	add_executable(${TARGET} EXCLUDE_FROM_ALL
		fuzz.cpp
		${ARGN}
	)
	set_target_properties(${TARGET} PROPERTIES OUTPUT_NAME ${EXE_NAME})

	target_link_libraries(${TARGET} server testutil rpcclient)

	add_dependencies(bitcoin-fuzzers ${TARGET})

	set_property(GLOBAL APPEND PROPERTY FUZZ_TARGETS ${TARGET})

	install_target(${TARGET}
		COMPONENT fuzzer
		EXCLUDE_FROM_ALL
	)
endmacro()

function(add_regular_fuzz_targets)
	foreach(_fuzz_test_name ${ARGN})
		sanitize_target_name("fuzz-" ${_fuzz_test_name} _fuzz_target_name)
		add_fuzz_target(
			${_fuzz_target_name}
			${_fuzz_test_name}

			# Sources
			"${_fuzz_test_name}.cpp"
		)
	endforeach()
endfunction()

include(SanitizeHelper)
function(add_deserialize_fuzz_targets)
	foreach(_fuzz_test_name ${ARGN})
		sanitize_target_name("fuzz-" ${_fuzz_test_name} _fuzz_target_name)
		add_fuzz_target(
			${_fuzz_target_name}
			${_fuzz_test_name}

			# Sources
			deserialize.cpp
		)

		sanitize_c_cxx_definition("" ${_fuzz_test_name} _target_definition)
		string(TOUPPER ${_target_definition} _target_definition)
		target_compile_definitions(${_fuzz_target_name} PRIVATE ${_target_definition})
	endforeach()
endfunction()

function(add_process_message_fuzz_targets)
	foreach(_fuzz_test_name ${ARGN})
		sanitize_target_name("fuzz-process_message_" ${_fuzz_test_name} _fuzz_target_name)
		add_fuzz_target(
			${_fuzz_target_name}
			process_message_${_fuzz_test_name}

			# Sources
			process_message.cpp
		)

		target_compile_definitions(${_fuzz_target_name} PRIVATE MESSAGE_TYPE=${_fuzz_test_name})
	endforeach()
endfunction()

add_regular_fuzz_targets(
	addrdb
	asmap
	base_encode_decode
	block
	bloom_filter
	rolling_bloom_filter
	cashaddr
	descriptor_parse
	eval_script
	hex
	integer
	net_permissions
	parse_hd_keypath
	parse_iso8601
	parse_numbers
	parse_script
	parse_univalue
	process_message
	psbt
	script
	script_flags
	spanparsing
	strprintf
	timedata
	transaction
	tx_in
	tx_out
)

add_deserialize_fuzz_targets(
	addr_info_deserialize
	address_deserialize
	addrman_deserialize
	banentry_deserialize
	block_deserialize
	block_file_info_deserialize
	block_filter_deserialize
	block_header_and_short_txids_deserialize
	blockheader_deserialize
	blocklocator_deserialize
	blockmerkleroot
	blocktransactions_deserialize
	blocktransactionsrequest_deserialize
	blockundo_deserialize
	bloomfilter_deserialize
	coins_deserialize
	diskblockindex_deserialize
	fee_rate_deserialize
	flat_file_pos_deserialize
	inv_deserialize
	key_origin_info_deserialize
	merkle_block_deserialize
	messageheader_deserialize
	netaddr_deserialize
	out_point_deserialize
	partial_merkle_tree_deserialize
	partially_signed_transaction_deserialize
	prefilled_transaction_deserialize
	psbt_input_deserialize
	psbt_output_deserialize
	pub_key_deserialize
	script_deserialize
	service_deserialize
	sub_net_deserialize
	tx_in_deserialize
	txoutcompressor_deserialize
	txundo_deserialize
)

add_process_message_fuzz_targets(
	addr
	block
	blocktxn
	cmpctblock
	feefilter
	filteradd
	filterclear
	filterload
	getaddr
	getblocks
	getblocktxn
	getdata
	getheaders
	headers
	inv
	mempool
	notfound
	ping
	pong
	sendcmpct
	sendheaders
	tx
	verack
	version
)
