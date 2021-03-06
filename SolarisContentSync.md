*_DEPRECATED, NO LONGER USED*_
# Specifications for Solaris Content Sync

## Goal:




The idea here is to be able to sync down the Solaris MPM(patches and patch clusters) content via satellite-sync from one satellite to another. This feature is exclusively for inter-spacewalk-sync.This includes ability to export the content through the exporter as well. The exporter needs the ability to look for solaris specific content, export it to an xml stream and sent it across to the destination server. The importer then reads the solaris xml stream, parses the content and puts the mpm files on to the file system mount point under /var/satellite and into the database.
## Requirements:



 * satellite-sync should be able to sync down the content from master servers

 * Support for importer to be able to import the Solaris mpm content as patches and path clusters.

 * Support for exporter to be able to extract the solaris specific data from the spacewalk database.

 * Handle caching similar to rpms.
## Technical Specifications :

### Changes to Importer: (_Estimate: 3 week_)

  


    * Need to make sure we bipass all the rpm, errata and kickstart logic and fork sat-sync to only deal with solaris specific content.

    * We need to add the necessary flags to turn on solaris support on satellite-syncs, possibly  --solaris.

     ` $ satellite-sync -c solaris-136540  `

    > Can't we determine the content type by channel arch vs the need to specify any special --solaris flag?  (tsanders)

So as todd said, we'll try to do this by some means to determine if its a solaris channel and handle the logic appropriately. ~~ Basically --content-type or mpm or --solaris ( not sure about the name yet), it will bipass current flow and fork(visualize some if checks) satsync to take a separate path for solaris specific content. ~~ This is to ensure that we dont conflict with existing rpm logic.

    * Hopefully caching scheme should already work for mpm as its by file basis luckly.

    * we need to add a import_solaris in sync_handler for the call to pass through to the top satsync layer from the importer.

   
    
          def import_solaris(batch):
              importer = solarisImporter.solarisImport(batch, diskImportLib.get_backend())
              importer.run()
    
        }}}
    
        * SolarisPackageContainer objects to create the necessary collection objects to be fed to the importer are already present for Source packages in sync_handler.SolarisPackageCollection.
    
         * Dumper classes for Solaris mpms need to be added to create objects necessary for caching and dumping the file on to the file system.
    {{{
        class SolarisMPMDumper(Dumper):
            _loader_class = xmlSource.SolarisMPMContainer
    
            def _getMethod(self):
                return self.dump.solarismpm
or similar.

    * In xmlWireSource, call the rpc layer fetchig the solaris content from the exporter.
    {{{
        def getSolChannelXmlStream(channel):
            return self._openSocketStream("dump.solChannels", (channel))

        def getMpm(self, solpkg, channel):
            """
             construct a sol compatible pkg structure and
             fetch the stream
            """
            return

     and few more additional call I see getting included as need araises
     }}}


    * Similar calls in xmlDiskSource for import from a dump

    * We can borrow some supporting library calls for mpm from rhnpush's rhn_mpm.py

    * We already ahve a mpmSource.py in the importer supporting the mpm content imports. We might need some additional changes to this, but mostly looks functional.

    * backendOracle.py should include the mappings for solaris tables.

    * backend.py's processPackage already takes into account solaris formats and tables. So we should be covered talking to the database layer.

    * We'll need another SolarisImporter.py class that handles interaction between satsync client bits, importer and db layer.
### Database tables of interest:



 * rhnSolarisPatch
 * rhnSolarisPatchPackages
 * rhnSolarisPatchSet
 * rhnSolarisPatchSetMembers
 * rhnSolarisPackage
### Exporter Changes:(_Estimate: 2 week_)
 


These are the changes to the master satellite's exporter module to export the content from database and send it across to the slaves.

    * satellite_exporter/exporter/dumper.py needs a new dumper class to handle solaris package content.

    {{{

        def dump_solaris_packages(self, packages)
            return self._sol_packages(packages, prefix='rhn-sol-package-',
                dump_class=SolPackagesDumper)

        def _sol_packages(packages, prefix, dump_class):
            """
            This is where we pass in the dump_class to the SatelliteDumper,
             dump the content from the exporter and write to xml"""

        see equivalent rpm packages dumper for hints.

       class SolPackagesDumper(CachedDumper, exportLib.SolPackagesDumper):
           def __init__(self, writer, pkgs):

               h = """query to fetch the solaris mpms,
                      patches and patch clusters"""
               CachedDumper.__init__(self, writer, statement=h, params=pkgs)

           def _get_key(self, params):
               return "xml-sol-packages/%s/rhn-sol-package-%s.xml" % (hash_val, package_id)
           def _dump_subelement(self, data):
               return exportLib.SolPackagesDumper.dump_subelement(self, data)

     }}}

    * satellite_exporter/exporter/exportLib.py needs a SolPackagesDumper class to extract the content from the database

     {{{

          class SolPackagesDumper(BaseDumper):
              tag_name = 'rhn-sol-packages'

              def set_iterator(self):
                  h = rhnSQL.prepare(""" Sample query for sol pkgs""")
                  h.execute()
                  return h

              def dump_subelement(self, data):
                  p = _SolPackageDumper(self._writer, data)
                  p.dump()

           class _SolPackageDumper(BaseRowDumper):
               tag_name = 'rhn-sol-packages'

               def set_attributes(self):
                   """ construct the attr necessary for xml"""
                   return attrdict
               def set_iterator(self):
                   """iterator to populate the xml"""
                   return ArrayIterator(arr)

       }}}

    * Now that we have all the business logic in place. Expose the rpc call so clients can call and download the xml stream. This should happen in non_auth_dumper.py

       {{{
           def get_sol_package(self, pckage, channel):
               return self._send_sol_package_stream(package, channel)

           def _send_sol_package_stream(package, channel):
               """ This should get the package path for solaris pkgs on filer
               and should jus pass the path to send_stream call"""
               path = self.get_sol_package_path_by_filename(package, channel)
               return self._send_stream(path)

        }}}
### rhn-satellite-Exporter Changes: (_Estimate: 1 week_)
 


    * We need to add the necessary flags to enable exports through the exporter tool.
    {{{ 
        $ rhn-satellite-exporter -c sol-test --dir /mnt/sol-dumps
    }}}

    * Changes to iss_ui.py will be necessary to support the solaris flags and step sequence functionality.

    * changes to dumper.py as well, should be same as the exporter changes above.


*_Total Dev estimate: 6 weeks *_

* Note: This content and code in this spec is just a boilerplate emphasising what and where the changes are required. This can change once the implementation begins.
### Use Cases:



1. Sync a solaris channel from master to slave satellite.
   $ satellite-sync --parent-sat=master-satellite.redhat.com -c <sol-channel-name> 

2. Export a solaris channel from a master or slave satellite
   $ rhn-satellite-export --dir /mnt/sol-dump -c <sol-channel-name> 
3. Sync a solaris channel from an exported dump
   $ satellite-sync -m /mnt/sol-dump -c <sol-channel-name> 
### Test Plans:

  * TODO

