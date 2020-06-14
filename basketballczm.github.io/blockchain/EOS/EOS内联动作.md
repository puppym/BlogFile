* https://developers.eos.io/eosio-home/docs/inline-actions
example:
//Only the addressbook account/contract can authorize this command. 
require_auth( name("addressbook"));

  [[eosio::action]]
  void notify(name user, std::string msg) {
    require_auth(get_self());
    require_recipient(user);
  }

[[eosio::action]]
  void upsert(name user, std::string first_name, std::string last_name, std::string street, std::string city, std::string state) {

    require_auth( user );

    address_index addresses(_code, _code.value);

    auto iterator = addresses.find(user.value);
    if( iterator == addresses.end() )
    {
      addresses.emplace(user, [&]( auto& row ) {
       row.key = user;
       row.first_name = first_name;
       row.last_name = last_name;
       row.street = street;
       row.city = city;
       row.state = state;
      });
      send_summary(user, "successfully emplaced record to addressbook");
    }
    else {
      std::string changes;
      addresses.modify(iterator, user, [&]( auto& row ) {
        row.key = user;
        row.first_name = first_name;
        row.last_name = last_name;
        row.street = street;
        row.city = city;
        row.state = state;
      });
      send_summary(user, "successfully modified record to addressbook");
    }
  }