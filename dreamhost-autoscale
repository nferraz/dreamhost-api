#!/usr/bin/env perl

use strict;
use warnings;

use lib "$ENV{HOME}/perl5/lib/perl5";

use Data::Dumper;
use Getopt::Long;
use Sys::Hostname;
use WWW::DreamHost::API;

my %opt = (
    minimum => 300,
    maximum => 600,
    margin  => 100,
    key => '',
    ps  => '',
);

GetOptions(
    \%opt,
    'minimum|min=i',
    'maximum|max=i',
    'margin=i',
    'key|k=s',
    'ps|hostname|host|p|h=s',
    'dry_run|dry',
);

my $API_KEY = $opt{key} || $ENV{DREAMHOST_API_KEY} || die "Please specify the API key\n";

my $ps = $opt{ps}                 ? $opt{ps}
       : hostname =~ /^ps[0-9]+$/ ? hostname
                                  : die "Please specify a hostname";

my $dreamhost = WWW::DreamHost::API->new($API_KEY);

my $used_memory       = used_memory();
my $configured_memory = configured_memory();

my $new_size = new_size( $used_memory, $configured_memory );

if ($opt{dry_run}) {
    print "configured: $configured_memory\nused: $used_memory\nnew size: $new_size\n";
    exit;
}

if ($new_size == $configured_memory) {
    warn "$0: Nothing to do!\n";
    exit;
}

my $rv = $dreamhost->command( 'dreamhost_ps-set_size', ps => $ps, size => $new_size );
check($rv);

# if everything goes ok, exit silently
exit;

sub check {
    my $res = shift;
    if ($res->{result} eq 'error') {
        die "Error: ", $res->{data}, "\n";
    }
}

sub new_size {
    my ( $used, $configured ) = @_;

    # new memory rounded to 50MB
    my $new = int(($used + $opt{margin}) / 50) * 50;

    # make sure it is within desired limits
    if ( $new < $opt{minimum} ) {
        $new = $opt{minimum};
    }
    elsif ( $new > $opt{maximum} ) {
        $new = $opt{maximum};
    }

    return $new;
}

sub used_memory {
    my $mem_str = `free|grep Mem`;

    #free         
    #             total       used       free     shared    buffers     cached
    #Mem:        307200     247056      60144          0          0      46476
    #-/+ buffers/cache:     200580     106620
    #Swap:            0          0          0

    my ($total,$used,$free) = $mem_str =~ m{Mem:\s+(\d+)\s+(\d+)\s+(\d+)};

    return int($used/1000);
}

sub configured_memory {
    my $size  = $dreamhost->command( 'dreamhost_ps-list_size_history', ps => $ps );
    check($size);

    my $configured_memory = $size->{data}->[-1]->{memory_mb};

    return $configured_memory;
}

__END__

=head1 NAME

dreamhost-autoscale - auto-scale dreamhost vps memory

=head1 DESCRIPTION

This tool was designed to auto-scale dreamhost virtual private servers, within
predetermined memory limits.

To be able to use it you need a DreamHost API key, which can be obtained here:

https://panel.dreamhost.com/index.cgi?tree=home.api

=head1 USAGE

    dreamhost-autoscale <options>

    # Example (using DreamHost's demo account):
    dreamhost-autoscale --key=6SHU5P2HLDAYECUM --ps=ps7093 --dry

=head1 OPTIONS

=over

=item --key

dreamhost api key

=item --ps

vps name (default: this hostname)

=item --min

minimum memory (default: 300)

=item --max

maximum memory (default: 600)

=item --margin

minimum free margin (default: 100)

=item --dry

dry-run; print what would be done but don't change anything

=back

=head1 DEPENDENCIES

WWW::DreamHost::API

=head1 AUTHOR

Nelson Ferraz <nferraz@gmail.com>

based on Serguei Trouchelle's WWW::DreamHost::API

=head1 LICENSE AND COPYRIGHT

This script is distributed under the same terms as Perl itself.

Copyright (c) 2012 Nelson Ferraz
